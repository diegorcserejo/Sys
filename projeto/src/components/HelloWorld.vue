<template>
  <div class="experience-container">
    <canvas ref="canvasRef" class="webgl-canvas"></canvas>
    <header class="header">
      <div class="logo">space<span class="logo-accent">edu</span></div>
      <nav class="nav-links">
        <button
          v-for="tab in tabs"
          :key="tab.id"
          class="nav-tab"
          :class="{ active: activeTab === tab.id }"
          @click="switchTab(tab.id)"
        >{{ tab.label }}</button>
      </nav>
      <button class="btn-enroll">Inscreva-se</button>
    </header>

    <!-- PLANETS / DWARF PLANETS SCROLL EXPERIENCE -->
    <div v-if="activeTab === 'planets' || activeTab === 'dwarfs'" :key="activeTab" class="planet-scroll">
      <section class="section hero-section">
        <div class="hero-content">
          <span class="subtitle">{{ activeTab === 'planets' ? 'PLANETA' : 'PLANETA ANÃO' }}</span>
          <h1 class="title">{{ currentPlanet.name }}</h1>
          <p class="description">{{ currentPlanet.heroDescription }}</p>
          <button class="btn-primary" @click="scrollToNextSection">COMEÇAR</button>
        </div>
        <button class="adjacent-planet left-planet" :aria-label="`Ver ${previousPlanet.name}`" @click="selectPlanet(-1)">
          <span class="planet-thumb" :style="thumbnailStyle(previousPlanet)"></span><span>{{ previousPlanet.name }}</span>
        </button>
        <button class="adjacent-planet right-planet" :aria-label="`Ver ${nextPlanet.name}`" @click="selectPlanet(1)">
          <span>{{ nextPlanet.name }}</span><span class="planet-thumb" :style="thumbnailStyle(nextPlanet)"></span>
        </button>
        <div class="scroll-indicator" @click="scrollToNextSection"><svg viewBox="0 0 24 24" width="24" height="24" stroke="currentColor" stroke-width="2" fill="none"><path d="M7 13l5 5 5-5M7 6l5 5 5-5" /></svg></div>
      </section>
      <section class="section detail-section">
        <div class="detail-content">
          <span class="tagline">{{ currentPlanet.tagline }}</span>
          <h2 class="detail-title">{{ currentPlanet.name }}</h2>
          <p class="detail-description">{{ currentPlanet.detailedInfo }}</p>
          <div class="actions">
            <button class="btn-primary" @click="selectPlanet(1)">PRÓXIMO {{ activeTab === 'planets' ? 'PLANETA' : 'CORPO' }}</button>
            <button class="btn-play" aria-label="Reproduzir vídeo"><svg viewBox="0 0 24 24" width="16" height="16" fill="currentColor"><polygon points="5 3 19 12 5 21 5 3" /></svg></button>
          </div>
        </div>
      </section>
      <section class="section surface-section">
        <aside class="planet-facts" aria-label="Dados do corpo celeste">
          <span class="facts-label">DOSSIÊ PLANETÁRIO</span>
          <dl>
            <div v-for="fact in currentPlanet.facts" :key="fact.label"><dt>{{ fact.label }}</dt><dd>{{ fact.value }}</dd></div>
          </dl>
        </aside>
        <div class="surface-content">
          <p class="surface-description">{{ currentPlanet.surfaceInfo }}</p>
        </div>
      </section>
    </div>

    <!-- STARS TAB -->
    <section v-else-if="activeTab === 'stars'" class="stars-panel">
      <div class="stars-header">
        <span class="tagline">CATÁLOGO ESTELAR</span>
        <h2 class="stars-title">Estrelas Notáveis</h2>
        <p class="stars-subtitle">Um guia das estrelas mais brilhantes e dos aglomerados estelares mais fascinantes do céu noturno, com suas cores e classificações reais.</p>
      </div>
      <h3 class="stars-group-title">✦ Estrelas Notáveis</h3>
      <div class="stars-grid">
        <article v-for="star in notableStars" :key="star.id" class="star-card">
          <div class="star-figure" :style="starFigureStyle(star)"></div>
          <span class="star-category star">Estrela</span>
          <h4 class="star-name">{{ star.name }}</h4>
          <p class="star-constellation">{{ star.constellation }}</p>
          <p class="star-description">{{ star.description }}</p>
          <div class="star-meta"><span>Mag {{ star.magnitude.toFixed(2) }}</span><span>{{ star.distance }}</span></div>
        </article>
      </div>
      <h3 class="stars-group-title">✦ Aglomerados Estelares</h3>
      <div class="stars-grid">
        <article v-for="star in starClusters" :key="star.id" class="star-card">
          <div class="star-figure" :style="starFigureStyle(star)"></div>
          <span class="star-category cluster">Aglomerado</span>
          <h4 class="star-name">{{ star.name }}</h4>
          <p class="star-constellation">{{ star.constellation }}</p>
          <p class="star-description">{{ star.description }}</p>
          <div class="star-meta"><span>Mag {{ star.magnitude.toFixed(2) }}</span><span>{{ star.distance }}</span></div>
        </article>
      </div>
    </section>

    <!-- SISTEMA TAB -->
    <section v-else class="system-panel">
      <div class="system-header">
        <span class="tagline">VISÃO GERAL</span>
        <h2 class="system-title">O Sistema Solar</h2>
        <p class="system-subtitle">Dados essenciais sobre a nossa vizinhança cósmica.</p>
      </div>
      <div class="system-grid">
        <article v-for="card in systemCards" :key="card.id" class="system-card">
          <span class="system-value">{{ card.value }}</span>
          <h4>{{ card.title }}</h4>
          <p>{{ card.description }}</p>
        </article>
      </div>
    </section>
  </div>
</template>

<script setup lang="ts">
import { computed, nextTick, onMounted, onUnmounted, ref } from 'vue'
import * as THREE from 'three'
import gsap from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
gsap.registerPlugin(ScrollTrigger)

type PlanetFact = { label: string; value: string }
type Planet = { id: string; name: string; tagline: string; heroDescription: string; detailedInfo: string; surfaceInfo: string; texture: string; facts: PlanetFact[] }
type Star = { id: string; name: string; constellation: string; category: 'ESTRELA' | 'AGLOMERADO'; description: string; color: string; magnitude: number; distance: string; cluster?: boolean }
type TabId = 'planets' | 'dwarfs' | 'stars' | 'system'

const tabs: { id: TabId; label: string }[] = [
  { id: 'planets', label: 'Planetas' },
  { id: 'dwarfs', label: 'Planetas Anões' },
  { id: 'stars', label: 'Estrelas' },
  { id: 'system', label: 'Sistema' }
]

const canvasRef = ref<HTMLCanvasElement | null>(null)
const activeTab = ref<TabId>('planets')
const planetIndex = ref(2)
const dwarfIndex = ref(0)
const textureBase = 'https://www.solarsystemscope.com/textures/download/'

const planetBlueprints = [
  ['mercure', 'MERCÚRIO', 'O PLANETA VELOZ', '2k_mercury.jpg', 'O menor planeta é um mundo rochoso coberto por antigas crateras de impacto. Sua atmosfera fina não retém calor, gerando enormes variações de temperatura entre o dia e a noite.'],
  ['venus', 'VÊNUS', 'O PLANETA ENCOBERTO', '2k_venus_surface.jpg', 'Vênus é envolvido por uma atmosfera densa de dióxido de carbono e nuvens brilhantes de ácido sulfúrico. Sob elas há uma superfície vulcânica sob pressão extrema.'],
  ['terre', 'TERRA', 'O PLANETA AZUL', 'https://threejs.org/examples/textures/planets/earth_atmos_2048.jpg', 'Nosso planeta é um mundo oceânico rochoso, com uma superfície ativa de montanhas, vales, planícies e cânions. A água líquida cobre cerca de 70% de sua superfície.'],
  ['mars', 'MARTE', 'O PLANETA VERMELHO', '2k_mars.jpg', 'Marte é um mundo desértico e frio, marcado por poeira rica em ferro, vulcões gigantes e cânions profundos. Indícios de rios e lagos antigos o tornam essencial na busca por habitabilidade passada.'],
  ['jupiter', 'JÚPITER', 'O PLANETA GIGANTE', '2k_jupiter.jpg', 'Júpiter é o maior planeta do Sistema Solar. Sua superfície visível é uma atmosfera turbulenta de hidrogênio e hélio, incluindo a duradoura Grande Mancha Vermelha.'],
  ['saturne', 'SATURNO', 'O PLANETA DOS ANÉIS', '2k_saturn.jpg', 'Saturno é um gigante gasoso reconhecido por seu sistema luminoso de anéis. Eles são formados principalmente por incontáveis fragmentos de gelo de água, de grãos de poeira a blocos enormes.'],
  ['uranus', 'URANO', 'O PLANETA DEITADO', '2k_uranus.jpg', 'Urano é um gigante de gelo com atmosfera azul-pálida colorida por metano. Sua inclinação axial extrema cria estações diferentes das de qualquer outro planeta.'],
  ['neptune', 'NETUNO', 'O PLANETA DOS VENTOS', '2k_neptune.jpg', 'Netuno é um gigante de gelo azul-profundo, distante do Sol. Sua atmosfera abriga os ventos mais rápidos já medidos no Sistema Solar.']
] as const
const dwarfBlueprints = [
  ['ceres', 'CERES', 'O ANÃO DO CINTURÃO', '2k_ceres.jpg', 'Ceres é o maior objeto do cinturão de asteroides e o único planeta anão do Sistema Solar interno. Sua superfície rochosa é coberta por crateras e possíveis depósitos de gelo de água.'],
  ['pluto', 'PLUTÃO', 'O REI DO CINTURÃO DE KUIPER', '2k_pluton.jpg', 'Plutão é um mundo gelado no Cinturão de Kuiper, famoso pelo coração de nitrogênio congelado, montanhas de gelo de água e uma atmosfera tênue que muda com as estações.'],
  ['haumea', 'HAUMEA', 'O ANÃO OVAL', '2k_haumea.jpg', 'Haumea é um planeta anão com forma alongada pela rotação extremamente rápida. Possui superfície brilhante de gelo de água e um sistema de anéis descoberto em 2017.'],
  ['makemake', 'MAKEMAKE', 'O ANÃO VERMELHO', '2k_makemake.jpg', 'Makemake é um mundo gelado e avermelhado no Cinturão de Kuiper, coberto por metano congelado, com uma pequena lua chamada MK 2.'],
  ['eris', 'ÉRIS', 'O ANÃO MAIS MASSIVO', '2k_eris.jpg', 'Éris é o planeta anão mais massivo conhecido, com superfície gelada e brilhante no disco disperso, bem além da órbita de Plutão.']
] as const
const planetName = (name: string) => name.charAt(0) + name.slice(1).toLowerCase()
const makePlanet = ([id, name, tagline, texture, surface]: typeof planetBlueprints[number]): Planet => ({ id, name, tagline, texture: texture.startsWith('http') ? texture : `${textureBase}${texture}`, surfaceInfo: surface, heroDescription: `Explore ${planetName(name)}, um dos oito planetas que orbitam o nosso Sol.`, detailedInfo: 'Carregando dados planetários...', facts: [{ label: 'FONTE DE DADOS', value: 'SOLAR SYSTEM OPEN DATA' }, { label: 'STATUS', value: 'SINCRONIZANDO' }] })
const makeDwarfPlanet = ([id, name, tagline, texture, surface]: typeof dwarfBlueprints[number]): Planet => ({ id, name, tagline, texture: texture.startsWith('http') ? texture : `${textureBase}${texture}`, surfaceInfo: surface, heroDescription: `Explore ${planetName(name)}, um dos cinco planetas anões reconhecidos do Sistema Solar.`, detailedInfo: 'Carregando dados planetários...', facts: [{ label: 'FONTE DE DADOS', value: 'SOLAR SYSTEM OPEN DATA' }, { label: 'STATUS', value: 'SINCRONIZANDO' }] })
const planets = ref<Planet[]>(planetBlueprints.map(makePlanet))
const dwarfPlanets = ref<Planet[]>(dwarfBlueprints.map(makeDwarfPlanet))

const currentIndex = computed(() => activeTab.value === 'planets' ? planetIndex.value : dwarfIndex.value)
const activePlanets = computed(() => activeTab.value === 'planets' ? planets.value : dwarfPlanets.value)
const currentPlanet = computed(() => activePlanets.value[currentIndex.value])
const previousPlanet = computed(() => activePlanets.value[(currentIndex.value + activePlanets.value.length - 1) % activePlanets.value.length])
const nextPlanet = computed(() => activePlanets.value[(currentIndex.value + 1) % activePlanets.value.length])

const stars: Star[] = [
  { id: 'spica', name: 'Spica', constellation: 'Virgem', category: 'ESTRELA', description: 'A estrela mais brilhante da constelação de Virgem.', color: '#9bb0ff', magnitude: 0.97, distance: '250 anos-luz' },
  { id: 'canopus', name: 'Canopus', constellation: 'Carina', category: 'ESTRELA', description: 'A segunda estrela mais brilhante do céu noturno, localizada na constelação de Carina.', color: '#fff4d6', magnitude: -0.74, distance: '310 anos-luz' },
  { id: 'acrux', name: 'Acrux', constellation: 'Cruzeiro do Sul', category: 'ESTRELA', description: 'A estrela principal (Alfa) da constelação do Cruzeiro do Sul.', color: '#a3b8ff', magnitude: 0.76, distance: '320 anos-luz' },
  { id: 'bellatrix', name: 'Bellatrix', constellation: 'Órion', category: 'ESTRELA', description: 'Uma das estrelas brilhantes que formam a constelação de Órion, conhecida como o "ombro esquerdo" do caçador.', color: '#9db4ff', magnitude: 1.64, distance: '250 anos-luz' },
  { id: 'achernar', name: 'Achernar', constellation: 'Erídano', category: 'ESTRELA', description: 'A estrela mais brilhante da constelação de Erídano.', color: '#b0c4ff', magnitude: 0.46, distance: '140 anos-luz' },
  { id: 'hadar', name: 'Hadar', constellation: 'Centauro', category: 'ESTRELA', description: 'Estrela proeminente da constelação de Centauro (Beta Centauri).', color: '#a3b8ff', magnitude: 0.61, distance: '390 anos-luz' },
  { id: 'adhara-mirzam', name: 'Adhara & Mirzam', constellation: 'Cão Maior', category: 'ESTRELA', description: 'Estrelas brilhantes da constelação do Cão Maior.', color: '#9db4ff', magnitude: 1.5, distance: '430 anos-luz' },
  { id: 'dubhe', name: 'Dubhe', constellation: 'Ursa Maior', category: 'ESTRELA', description: 'Uma das estrelas principais da Ursa Maior.', color: '#ffab6b', magnitude: 1.79, distance: '120 anos-luz' },
  { id: 'elnath', name: 'Elnath', constellation: 'Touro / Cocheiro', category: 'ESTRELA', description: 'Estrela na fronteira entre as constelações de Touro e Cocheiro.', color: '#b0c4ff', magnitude: 1.65, distance: '130 anos-luz' },
  { id: 'alphard', name: 'Alphard', constellation: 'Hidra', category: 'ESTRELA', description: 'A estrela mais brilhante da constelação de Hidra.', color: '#ffb46e', magnitude: 1.98, distance: '180 anos-luz' },
  { id: 'mirach', name: 'Mirach', constellation: 'Andrômeda', category: 'ESTRELA', description: 'Estrela da constelação de Andrômeda.', color: '#ff7b6e', magnitude: 2.05, distance: '200 anos-luz' },
  { id: 'peacock', name: 'Peacock', constellation: 'Pavão', category: 'ESTRELA', description: 'Estrela principal da constelação de Pavão.', color: '#a3b8ff', magnitude: 1.94, distance: '180 anos-luz' },
  { id: 'atria', name: 'Atria', constellation: 'Triângulo Austral', category: 'ESTRELA', description: 'Estrela principal da constelação de Triângulo Austral.', color: '#ffb46e', magnitude: 1.91, distance: '390 anos-luz' },
  { id: 'sargas', name: 'Sargas', constellation: 'Escorpião', category: 'ESTRELA', description: 'Estrela principal da constelação de Escorpião.', color: '#fff8d8', magnitude: 1.87, distance: '300 anos-luz' },
  { id: 'kaus-australis', name: 'Kaus Australis', constellation: 'Sagitário', category: 'ESTRELA', description: 'Estrela principal da constelação de Sagitário.', color: '#b0c4ff', magnitude: 1.85, distance: '140 anos-luz' },
  { id: 'pleiades', name: 'Pleiades (Plêiades)', constellation: 'Touro', category: 'AGLOMERADO', description: 'Um famoso aglomerado estelar aberto localizado na constelação de Touro. Visível a olho nu, contém centenas de estrelas jovens e azuladas.', color: '#a3b8ff', magnitude: 1.6, distance: '440 anos-luz', cluster: true },
  { id: 'messier-46', name: 'Messier 46', constellation: 'Popa (Puppis)', category: 'AGLOMERADO', description: 'Um aglomerado aberto de estrelas situado na constelação de Popa (Puppis). Contém cerca de 500 estrelas e uma nebulosa planetária.', color: '#ffe4a1', magnitude: 6.1, distance: '5.400 anos-luz', cluster: true }
]
const notableStars = computed(() => stars.filter((s) => s.category === 'ESTRELA'))
const starClusters = computed(() => stars.filter((s) => s.category === 'AGLOMERADO'))

const systemCards = [
  { id: 'sun', title: 'O Sol', value: '1 estrela', description: 'Estrela anã amarela de classe G2V que contém 99,8% da massa total e fornece energia para todo o sistema.' },
  { id: 'planets', title: 'Planetas', value: '8 mundos', description: 'Mercúrio, Vênus, Terra, Marte, Júpiter, Saturno, Urano e Netuno — quatro rochosos internos e quatro gigantes externos.' },
  { id: 'dwarfs', title: 'Planetas Anões', value: '5 corpos', description: 'Ceres, Plutão, Haumea, Makemake e Éris — corpos que orbitam o Sol mas não limparam sua vizinhança orbital.' },
  { id: 'moons', title: 'Luas', value: '290+', description: 'Saturno lidera com 146 luas confirmadas, seguido de Júpiter com 95. A Terra possui 1 lua e Marte 2 pequenas.' },
  { id: 'asteroid', title: 'Cinturão de Asteroides', value: 'Entre Marte e Júpiter', description: 'Contém mais de 1 milhão de asteroides maiores que 1 km, com Ceres como o seu maior integrante.' },
  { id: 'kuiper', title: 'Cinturão de Kuiper', value: 'Além de Netuno', description: 'Região de corpos gelados de 30 a 55 UA do Sol, que inclui Plutão, Haumea e Makemake.' },
  { id: 'oort', title: 'Nuvem de Oort', value: 'Fronteira do sistema', description: 'Reservatório esférico de cometas a até 100.000 UA do Sol, fonte dos cometas de longo período.' },
  { id: 'age', title: 'Idade e Formação', value: '4,6 bilhões de anos', description: 'Formado a partir de uma nuvem molecular em colapso. O Sol surgiu no centro e os planetas no disco de acreção.' }
]

let scene: THREE.Scene, camera: THREE.PerspectiveCamera, renderer: THREE.WebGLRenderer, planetMesh: THREE.Mesh<THREE.SphereGeometry, THREE.MeshStandardMaterial>, atmosphereMesh: THREE.Mesh<THREE.SphereGeometry, THREE.ShaderMaterial>, cloudMesh: THREE.Mesh<THREE.SphereGeometry, THREE.MeshPhongMaterial>, ringMesh: THREE.Mesh<THREE.RingGeometry, THREE.MeshBasicMaterial>, moonOrbit: THREE.Group, moonMesh: THREE.Mesh<THREE.SphereGeometry, THREE.MeshStandardMaterial>, sunMesh: THREE.Mesh<THREE.SphereGeometry, THREE.ShaderMaterial>, animationFrameId = 0
let scrollTimeline: gsap.core.Timeline | undefined
let sunCorona: THREE.Sprite | null = null
const sunUniforms = { uTime: { value: 0 } }
const textureLoader = new THREE.TextureLoader()
const surfaceRelief: Record<string, number> = { mercure: .18, venus: .1, terre: .22, mars: .28, ceres: .2, jupiter: .09, saturne: .08, uranus: .04, neptune: .05, pluto: .15, haumea: .12, makemake: .18, eris: .15 }
const fallbackColors: Record<string, string> = { mercure: '#7c7268', venus: '#b47742', terre: '#1d5687', mars: '#a04428', ceres: '#8f8478', jupiter: '#c48b5d', saturne: '#d6bc8a', uranus: '#8ac8d4', neptune: '#315da2', pluto: '#c4ad93', haumea: '#a08b78', makemake: '#b0805c', eris: '#bfc0c5' }
const planetMotion: Record<string, { spin: number; tilt: number; scale: number }> = { mercure: { spin: .0028, tilt: .03, scale: .86 }, venus: { spin: .00055, tilt: 3.09, scale: 1.02 }, terre: { spin: .0015, tilt: .41, scale: 1 }, mars: { spin: .0014, tilt: .44, scale: .9 }, ceres: { spin: .0031, tilt: .03, scale: .55 }, jupiter: { spin: .0032, tilt: .05, scale: 1.25 }, saturne: { spin: .0026, tilt: .47, scale: 1.15 }, uranus: { spin: .0011, tilt: 1.71, scale: 1.03 }, neptune: { spin: .002, tilt: .49, scale: 1.04 }, pluto: { spin: .0013, tilt: 2.17, scale: .52 }, haumea: { spin: .0042, tilt: 1.32, scale: .55 }, makemake: { spin: .0028, tilt: .2, scale: .52 }, eris: { spin: .0012, tilt: .15, scale: .55 } }
let activeMotion = planetMotion.terre
let textureRequestId = 0

const fallbackTexture = (color: string) => {
  const canvas = document.createElement('canvas'); canvas.width = canvas.height = 512
  const context = canvas.getContext('2d')!
  const gradient = context.createRadialGradient(256, 256, 0, 256, 256, 256)
  gradient.addColorStop(0, color); gradient.addColorStop(1, '#000000')
  context.fillStyle = gradient; context.fillRect(0, 0, 512, 512)
  for (let i = 0; i < 3000; i++) { context.fillStyle = `rgba(255,255,255,${Math.random() * .12})`; context.beginPath(); context.arc(Math.random() * 512, Math.random() * 512, Math.random() * 2.5, 0, Math.PI * 2); context.fill() }
  const texture = new THREE.CanvasTexture(canvas); texture.colorSpace = THREE.SRGBColorSpace; return texture
}
const loadPlanetTexture = (planet: Planet) => {
  const requestId = ++textureRequestId
  textureLoader.load(planet.texture, (texture) => {
    if (requestId !== textureRequestId) { texture.dispose(); return }
    texture.colorSpace = THREE.SRGBColorSpace
    planetMesh.material.map?.dispose()
    planetMesh.material.map = texture
    planetMesh.material.bumpMap = texture
    planetMesh.material.emissiveMap = texture
    planetMesh.material.bumpScale = surfaceRelief[planet.id]
    planetMesh.material.needsUpdate = true
  }, undefined, () => {
    if (requestId !== textureRequestId) return
    planetMesh.material.map?.dispose()
    planetMesh.material.map = fallbackTexture(fallbackColors[planet.id])
    planetMesh.material.bumpMap = null
    planetMesh.material.emissiveMap = null
    planetMesh.material.needsUpdate = true
  })
}
const thumbnailStyle = (planet: Planet) => ({ backgroundImage: `url("${planet.texture}")` })
const updatePlanetMotion = (planet: Planet) => {
  activeMotion = planetMotion[planet.id]
  gsap.to(planetMesh.scale, { x: activeMotion.scale, y: activeMotion.scale, z: activeMotion.scale, duration: .7, ease: 'power2.out', overwrite: true })
  gsap.to(planetMesh.rotation, { z: activeMotion.tilt, duration: .85, ease: 'power2.out', overwrite: 'auto' })
}
const updatePlanetFeatures = (planet: Planet) => {
  atmosphereMesh.visible = ['terre', 'mars', 'jupiter', 'saturne', 'uranus', 'neptune'].includes(planet.id)
  cloudMesh.visible = planet.id === 'terre'
  ringMesh.visible = ['saturne', 'haumea'].includes(planet.id)
  moonOrbit.visible = ['terre', 'pluto'].includes(planet.id)
}
const selectPlanet = (delta: number) => {
  const list = activePlanets.value
  const next = (currentIndex.value + delta + list.length) % list.length
  if (activeTab.value === 'planets') planetIndex.value = next
  else dwarfIndex.value = next
  if (planetMesh) {
    loadPlanetTexture(currentPlanet.value)
    updatePlanetFeatures(currentPlanet.value)
    updatePlanetMotion(currentPlanet.value)
  }
}
const scrollToNextSection = () => {
  if (activeTab.value === 'planets' || activeTab.value === 'dwarfs') {
    window.scrollTo({ top: window.scrollY + window.innerHeight, behavior: 'smooth' })
  }
}
const killScrollAnimations = () => {
  scrollTimeline?.kill(); scrollTimeline = undefined
  ScrollTrigger.getAll().forEach((trigger) => trigger.kill())
}
const setupScrollAnimations = () => {
  if (!planetMesh || activeTab.value === 'stars' || activeTab.value === 'system') return
  killScrollAnimations()
  gsap.set(planetMesh.position, { x: 0, y: -1.8, z: 1.5 })
  gsap.set(camera.position, { x: 0, y: 0, z: 6 })
  const heroContent = document.querySelector('.hero-content')
  const detailContent = document.querySelector('.detail-content')
  const surfaceContent = document.querySelector('.surface-content')
  const planetFacts = document.querySelector('.planet-facts')
  const adjacentButtons = document.querySelectorAll('.adjacent-planet')
  const scrollIndicator = document.querySelector('.scroll-indicator')
  if (!heroContent || !detailContent || !surfaceContent || !planetFacts) return
  gsap.set([detailContent, surfaceContent, planetFacts], { opacity: 0 })
  window.scrollTo(0, 0)
  scrollTimeline = gsap.timeline({ scrollTrigger: { trigger: '.planet-scroll', start: 'top top', end: 'bottom bottom', scrub: 1.2 } })
  scrollTimeline
    .to(planetMesh.position, { x: 1.8, y: .1, z: 2.2, ease: 'power1.inOut' }, 0)
    .to(planetMesh.rotation, { x: .2, y: Math.PI * .8, ease: 'none' }, 0)
    .to(heroContent, { opacity: 0, y: -50 }, 0)
    .to(adjacentButtons, { opacity: 0, y: -50, stagger: .05 }, 0)
    .to(scrollIndicator, { opacity: 0, y: -50 }, 0)
    .fromTo(detailContent, { opacity: 0, x: -80 }, { opacity: 1, x: 0, ease: 'power2.out' }, .3)
    .to(camera.position, { z: 3.2, y: .2, ease: 'power2.inOut' }, 1)
    .to(planetMesh.position, { x: 0, y: .5, z: 1.8, ease: 'power2.inOut' }, 1)
    .to(detailContent, { opacity: 0, y: -30 }, 1)
    .fromTo(surfaceContent, { opacity: 0, y: 50 }, { opacity: 1, y: 0, ease: 'power2.out' }, 1.4)
    .fromTo(planetFacts, { opacity: 0, y: 30 }, { opacity: 1, y: 0, ease: 'power2.out' }, 1.55)
}
const switchTab = async (tab: TabId) => {
  if (activeTab.value === tab) return
  const wasPlanetTab = activeTab.value === 'planets' || activeTab.value === 'dwarfs'
  activeTab.value = tab
  window.scrollTo(0, 0)
  if (tab === 'planets' || tab === 'dwarfs') {
    planetMesh.visible = true
    sunMesh.visible = true
    loadPlanetTexture(currentPlanet.value)
    updatePlanetFeatures(currentPlanet.value)
    updatePlanetMotion(currentPlanet.value)
    await nextTick()
    setupScrollAnimations()
  } else if (tab === 'stars') {
    planetMesh.visible = false
    sunMesh.visible = false
    if (wasPlanetTab) killScrollAnimations()
    gsap.from('.star-card', { opacity: 0, y: 26, stagger: .035, duration: .45, ease: 'power2.out' })
  } else {
    planetMesh.visible = false
    sunMesh.visible = true
    if (wasPlanetTab) killScrollAnimations()
    gsap.from('.system-card', { opacity: 0, y: 26, stagger: .06, duration: .45, ease: 'power2.out' })
  }
}
const starFigureStyle = (star: Star) => {
  const color = star.color
  const delay = (star.id.length * .18).toFixed(2)
  if (star.cluster) {
    return {
      width: '80px', height: '80px',
      animationDelay: `${delay}s`,
      background: `radial-gradient(circle at 30% 38%, #fff 0%, ${color} 22%, transparent 54%),radial-gradient(circle at 65% 25%, #fff 0%, ${color} 17%, transparent 45%),radial-gradient(circle at 48% 64%, #fff 0%, ${color} 24%, transparent 58%),radial-gradient(circle at 78% 66%, #fff 0%, ${color} 15%, transparent 42%),radial-gradient(circle at 18% 74%, #fff 0%, ${color} 13%, transparent 38%),radial-gradient(circle at 55% 18%, ${color}22 0%, transparent 32%),radial-gradient(circle at 42% 82%, ${color}22 0%, transparent 32%)`,
      boxShadow: `0 0 32px ${color}55`
    }
  }
  if (star.id === 'adhara-mirzam') {
    return {
      width: '86px', height: '86px',
      animationDelay: `${delay}s`,
      background: `radial-gradient(circle at 36% 42%, #fff 0%, ${color} 26%, transparent 54%),radial-gradient(circle at 66% 60%, #fff 0%, #b5c7ff 20%, transparent 46%),radial-gradient(circle at 30% 45%, ${color}22 0%, transparent 45%)`,
      boxShadow: `0 0 28px ${color}66`
    }
  }
  const size = Math.max(40, Math.min(76, 62 - star.magnitude * 9))
  return {
    width: `${size}px`, height: `${size}px`,
    animationDelay: `${delay}s`,
    background: `radial-gradient(circle at 35% 35%, #fff 0%, ${color} 30%, ${color}66 55%, transparent 76%)`,
    boxShadow: `0 0 16px ${color}, 0 0 42px ${color}44, inset 0 0 10px #ffffff33`
  }
}

const loadPlanetData = async () => {
  try {
    const [planetResponses, dwarfResponses] = await Promise.all([
      Promise.all(planetBlueprints.map(async ([id]) => { const response = await fetch(`https://api.le-systeme-solaire.net/rest/bodies/${id}`); if (!response.ok) throw new Error(id); return response.json() })),
      Promise.all(dwarfBlueprints.map(async ([id]) => { const response = await fetch(`https://api.le-systeme-solaire.net/rest/bodies/${id}`); if (!response.ok) throw new Error(id); return response.json() }))
    ])
    const enrich = (blueprint: typeof planetBlueprints[number] | typeof dwarfBlueprints[number], data: any, makeFn: (b: any) => Planet): Planet => {
      const planet = makeFn(blueprint)
      const radius = new Intl.NumberFormat('pt-BR').format(Math.round(data.meanRadius))
      const moons = data.moons?.length ?? 0
      const distance = new Intl.NumberFormat('pt-BR').format(Math.round(data.semimajorAxis))
      const mass = data.mass ? `${data.mass.massValue} x 10^${data.mass.massExponent} kg` : 'N/D'
      const temperature = data.avgTemp ? `${Math.round(data.avgTemp - 273.15)} C` : 'N/D'
      return { ...planet, detailedInfo: `${planetName(planet.name)} tem raio médio de ${radius} km, gravidade de superfície de ${Number(data.gravity).toFixed(2)} m/s2 e ${moons} lua${moons === 1 ? ' conhecida' : 's conhecidas'}. Ele completa uma órbita ao redor do Sol em aproximadamente ${Number(data.sideralOrbit).toFixed(1)} dias terrestres.`, facts: [{ label: 'RAIO MÉDIO', value: `${radius} km` }, { label: 'MASSA', value: mass }, { label: 'DISTÂNCIA ORBITAL', value: `${distance} km` }, { label: 'TEMPERATURA MÉDIA', value: temperature }, { label: 'VELOCIDADE DE ESCAPE', value: `${Number(data.escape).toFixed(1)} km/s` }, { label: 'INCLINAÇÃO AXIAL', value: `${Number(data.axialTilt).toFixed(1)} graus` }] }
    }
    planets.value = planetBlueprints.map((blueprint, index) => enrich(blueprint, planetResponses[index], makePlanet))
    dwarfPlanets.value = dwarfBlueprints.map((blueprint, index) => enrich(blueprint, dwarfResponses[index], makeDwarfPlanet))
  } catch {
    const fallback = (planet: Planet): Planet => ({ ...planet, detailedInfo: `${planetName(planet.name)} é um corpo celeste do Sistema Solar. Explore este mundo por sua textura de superfície e retorne mais tarde para os dados ao vivo.`, facts: [{ label: 'CATÁLOGO', value: 'SISTEMA SOLAR' }, { label: 'TIPO', value: 'CORPO PLANETÁRIO' }, { label: 'STATUS DOS DADOS', value: 'CACHE OFFLINE' }] })
    planets.value = planets.value.map(fallback)
    dwarfPlanets.value = dwarfPlanets.value.map(fallback)
  }
}

const initThree = () => {
  if (!canvasRef.value) return
  scene = new THREE.Scene(); scene.fog = new THREE.FogExp2(0x050811, .05)
  camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, .1, 100); camera.position.set(0, 0, 6)
  renderer = new THREE.WebGLRenderer({ canvas: canvasRef.value, antialias: true, alpha: true }); renderer.setSize(window.innerWidth, window.innerHeight); renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2)); renderer.toneMapping = THREE.ACESFilmicToneMapping; renderer.outputColorSpace = THREE.SRGBColorSpace
  planetMesh = new THREE.Mesh(new THREE.SphereGeometry(2, 96, 96), new THREE.MeshStandardMaterial({ map: fallbackTexture('#244f79'), roughness: .72, metalness: .01 })); planetMesh.position.set(0, -1.8, 1.5); scene.add(planetMesh)
  atmosphereMesh = new THREE.Mesh(new THREE.SphereGeometry(2.06, 96, 96), new THREE.ShaderMaterial({ transparent: true, side: THREE.BackSide, depthWrite: false, blending: THREE.AdditiveBlending, vertexShader: 'varying vec3 vNormal; void main() { vNormal = normalize(normalMatrix * normal); gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0); }', fragmentShader: 'varying vec3 vNormal; void main() { float rim = pow(0.72 - dot(vNormal, vec3(0.0, 0.0, 1.0)), 3.0); gl_FragColor = vec4(0.18, 0.55, 1.0, rim * 0.32); }' })); planetMesh.add(atmosphereMesh)
  const cloudTexture = textureLoader.load('https://threejs.org/examples/textures/planets/earth_clouds_1024.png', (texture) => { texture.colorSpace = THREE.SRGBColorSpace }, undefined, () => { cloudMesh.visible = false }); cloudMesh = new THREE.Mesh(new THREE.SphereGeometry(2.025, 96, 96), new THREE.MeshPhongMaterial({ map: cloudTexture, transparent: true, opacity: .38, depthWrite: false })); planetMesh.add(cloudMesh)
  const ringTexture = textureLoader.load(`${textureBase}2k_saturn_ring_alpha.png`); ringTexture.colorSpace = THREE.SRGBColorSpace; ringMesh = new THREE.Mesh(new THREE.RingGeometry(2.28, 3.45, 128), new THREE.MeshBasicMaterial({ map: ringTexture, alphaMap: ringTexture, color: 0xe8d5ae, transparent: true, opacity: .82, alphaTest: .06, side: THREE.DoubleSide, depthWrite: false })); ringMesh.rotation.x = Math.PI / 2; ringMesh.visible = false; planetMesh.add(ringMesh)
  moonOrbit = new THREE.Group(); moonOrbit.rotation.z = .2; moonMesh = new THREE.Mesh(new THREE.SphereGeometry(.42, 64, 64), new THREE.MeshStandardMaterial({ roughness: .95, metalness: 0 })); moonMesh.position.set(3.2, .15, 0); textureLoader.load('https://threejs.org/examples/textures/planets/moon_1024.jpg', (texture) => { texture.colorSpace = THREE.SRGBColorSpace; moonMesh.material.map = texture; moonMesh.material.needsUpdate = true }); moonOrbit.add(moonMesh); planetMesh.add(moonOrbit)
  const sunVertexShader = 'varying vec2 vUv; void main() { vUv = uv; gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0); }'
  const sunFragmentShader = `uniform float uTime; varying vec2 vUv; float hash(vec2 p){ return fract(sin(dot(p, vec2(127.1,311.7))) * 43758.5453123); } float noise(vec2 p){ vec2 i=floor(p), f=fract(p); f=f*f*(3.0-2.0*f); return mix(mix(hash(i),hash(i+vec2(1.,0.)),f.x), mix(hash(i+vec2(0.,1.)),hash(i+vec2(1.,1.)),f.x), f.y); } float fbm(vec2 p){ float v=0., a=.5; for(int i=0;i<5;i++){ v+=a*noise(p); p*=2.07; a*=.5; } return v; } void main(){ float n1=fbm(vUv*7.+uTime*.12); float n2=fbm(vUv*13.-uTime*.08+4.7); float n3=fbm(vUv*22.+uTime*.04+9.3); float p=n1*.55+n2*.3+n3*.15; vec3 core=vec3(1.,.96,.78); vec3 mid=vec3(1.,.68,.22); vec3 dark=vec3(.82,.38,.06); vec3 c=mix(dark,mid,p); c=mix(c,core,smoothstep(.5,1.,p)); float limb=1.-smoothstep(.5,1.,length(vUv-.5)*1.55); c*=.72+.28*limb; gl_FragColor=vec4(c,1.); }`
  sunMesh = new THREE.Mesh(new THREE.SphereGeometry(1.18, 96, 96), new THREE.ShaderMaterial({ uniforms: sunUniforms, vertexShader: sunVertexShader, fragmentShader: sunFragmentShader })); sunMesh.position.set(-7.2, 3.8, -6); scene.add(sunMesh)
  const coronaCanvas = document.createElement('canvas'); coronaCanvas.width = coronaCanvas.height = 512
  const coronaCtx = coronaCanvas.getContext('2d')!
  const coronaGrad = coronaCtx.createRadialGradient(256, 256, 0, 256, 256, 256)
  coronaGrad.addColorStop(0, 'rgba(255, 190, 80, 1)')
  coronaGrad.addColorStop(.16, 'rgba(255, 160, 50, .6)')
  coronaGrad.addColorStop(.36, 'rgba(255, 120, 25, .22)')
  coronaGrad.addColorStop(.6, 'rgba(255, 95, 15, .07)')
  coronaGrad.addColorStop(.82, 'rgba(255, 85, 10, .025)')
  coronaGrad.addColorStop(1, 'rgba(255, 80, 5, 0)')
  coronaCtx.fillStyle = coronaGrad; coronaCtx.fillRect(0, 0, 512, 512)
  const coronaTexture = new THREE.CanvasTexture(coronaCanvas); coronaTexture.colorSpace = THREE.SRGBColorSpace
  sunCorona = new THREE.Sprite(new THREE.SpriteMaterial({ map: coronaTexture, blending: THREE.AdditiveBlending, depthWrite: false, transparent: true }))
  sunCorona.scale.set(9, 9, 1); sunCorona.position.copy(sunMesh.position); scene.add(sunCorona)
  updatePlanetMotion(currentPlanet.value)
  loadPlanetTexture(currentPlanet.value)
  updatePlanetFeatures(currentPlanet.value)
  const geometry = new THREE.BufferGeometry(), positions = new Float32Array(3600); for (let i = 0; i < positions.length; i += 3) { positions[i] = (Math.random() - .5) * 50; positions[i + 1] = (Math.random() - .5) * 50; positions[i + 2] = -Math.random() * 30 }; geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3)); scene.add(new THREE.Points(geometry, new THREE.PointsMaterial({ color: 0xffffff, size: .05, transparent: true, opacity: .8 })))
  scene.add(new THREE.AmbientLight(0xffffff, 1.8)); const sunlight = new THREE.DirectionalLight(0xffffff, 1.1); sunlight.position.copy(sunMesh.position); scene.add(sunlight)
  const animate = () => { planetMesh.rotation.y += activeMotion.spin; cloudMesh.rotation.y += .00055; ringMesh.rotation.z += .00035; moonOrbit.rotation.y += .006; moonMesh.rotation.y += .001; sunMesh.rotation.y += .0007; sunUniforms.uTime.value += .016; renderer.render(scene, camera); animationFrameId = requestAnimationFrame(animate) }; animate()
  setupScrollAnimations()
}
const onWindowResize = () => { if (!camera || !renderer) return; camera.aspect = window.innerWidth / window.innerHeight; camera.updateProjectionMatrix(); renderer.setSize(window.innerWidth, window.innerHeight) }
onMounted(async () => { initThree(); window.addEventListener('resize', onWindowResize); await loadPlanetData() })
onUnmounted(() => { window.removeEventListener('resize', onWindowResize); cancelAnimationFrame(animationFrameId); killScrollAnimations(); sunCorona?.material.map?.dispose(); sunCorona?.material.dispose(); sunMesh?.geometry.dispose(); sunMesh?.material.dispose(); moonMesh?.geometry.dispose(); moonMesh?.material.map?.dispose(); moonMesh?.material.dispose(); ringMesh?.geometry.dispose(); ringMesh?.material.map?.dispose(); ringMesh?.material.dispose(); cloudMesh?.geometry.dispose(); cloudMesh?.material.map?.dispose(); cloudMesh?.material.dispose(); atmosphereMesh?.geometry.dispose(); atmosphereMesh?.material.dispose(); planetMesh?.geometry.dispose(); planetMesh?.material.map?.dispose(); planetMesh?.material.dispose(); renderer?.dispose() })
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Cinzel:wght@500;700;800&family=Plus+Jakarta+Sans:wght@300;400;600&display=swap');
.experience-container{position:relative;width:100%;min-height:100vh;background:#050811;color:#fff;font-family:'Plus Jakarta Sans',sans-serif;overflow-x:hidden}
.webgl-canvas{position:fixed;inset:0;width:100vw;height:100vh;z-index:1;pointer-events:none}
.header{position:fixed;top:0;left:0;width:100%;display:flex;justify-content:space-between;align-items:center;padding:2rem 4rem;z-index:10;box-sizing:border-box}
.logo{font-size:1.5rem;font-weight:700;letter-spacing:1px}.logo-accent{color:#63b3ed}
.nav-links{display:flex;gap:1.6rem;align-items:center}
.nav-tab{background:transparent;border:0;color:rgba(255,255,255,.6);font:inherit;font-size:.9rem;letter-spacing:1px;padding:.4rem .9rem;border-radius:8px;cursor:pointer;transition:color .3s,background .3s}
.nav-tab:hover{color:#fff}
.nav-tab.active{color:#63b3ed;background:rgba(99,179,237,.1)}
.btn-enroll{background:rgba(255,255,255,.12);backdrop-filter:blur(10px);border:1px solid rgba(255,255,255,.2);color:#fff;padding:.6rem 1.8rem;border-radius:50px;cursor:pointer;font-weight:600}

/* ===== PLANET SCROLL EXPERIENCE ===== */
.planet-scroll{position:relative;z-index:2}
.section{position:relative;height:100vh;width:100%;display:flex;align-items:center;padding:0 6rem;box-sizing:border-box}
.hero-section{flex-direction:column;justify-content:center;text-align:center}
.hero-content{max-width:650px;margin-top:-5vh;text-shadow:0 2px 24px rgba(0,0,0,.55)}
.subtitle{font-size:.9rem;letter-spacing:6px;color:rgba(255,255,255,.6);display:block;margin-bottom:.5rem}
.title{font-family:'Cinzel',serif;font-size:5.5rem;font-weight:700;letter-spacing:8px;margin:0 0 1.5rem}
.description{font-size:.95rem;line-height:1.8;color:rgba(255,255,255,.7);margin-bottom:2.5rem;max-width:650px}
.btn-primary{background:#fff;color:#050811;border:0;padding:.9rem 2.2rem;border-radius:50px;font-size:.85rem;font-weight:700;letter-spacing:1.5px;cursor:pointer;transition:transform .3s,box-shadow .3s}
.btn-primary:hover{transform:translateY(-2px);box-shadow:0 10px 25px rgba(255,255,255,.2)}
.adjacent-planet{position:absolute;top:55%;display:flex;align-items:center;gap:1rem;padding:0;border:0;background:transparent;font:inherit;font-size:.8rem;letter-spacing:3px;color:rgba(255,255,255,.6);cursor:pointer;transition:color .3s;z-index:4}
.adjacent-planet:hover{color:#fff}
.left-planet{left:5rem}
.right-planet{right:5rem;flex-direction:row-reverse}
.planet-thumb{width:45px;height:45px;flex:0 0 45px;border-radius:50%;background-position:center;background-size:cover;box-shadow:0 0 15px rgba(255,255,255,.1)}
.scroll-indicator{position:absolute;bottom:2.5rem;cursor:pointer;animation:bounce 2s infinite;opacity:.7;z-index:4}
@keyframes bounce{0%,20%,50%,80%,100%{transform:translateY(0)}40%{transform:translateY(-8px)}60%{transform:translateY(-4px)}}
.detail-section{justify-content:flex-start}
.detail-content{max-width:480px}
.tagline{font-size:.85rem;letter-spacing:4px;color:rgba(255,255,255,.5);display:block;margin-bottom:.5rem}
.detail-title{font-family:'Cinzel',serif;font-size:4rem;letter-spacing:6px;margin:0 0 1.5rem}
.detail-description{font-size:.95rem;line-height:1.8;color:rgba(255,255,255,.7);margin-bottom:2rem;max-width:480px}
.actions{display:flex;align-items:center;gap:1.5rem}
.btn-play{width:48px;height:48px;border-radius:50%;background:rgba(255,255,255,.1);backdrop-filter:blur(10px);border:1px solid rgba(255,255,255,.2);color:#fff;display:flex;align-items:center;justify-content:center;cursor:pointer}
.surface-section{justify-content:flex-end;align-items:flex-end;padding-bottom:8rem}
.surface-content{max-width:420px;background:rgba(5,8,17,.6);backdrop-filter:blur(16px);padding:2rem;border-radius:16px;border:1px solid rgba(255,255,255,.08)}
.surface-description{color:rgba(255,255,255,.8);margin:0;font-size:.95rem;line-height:1.8}
.planet-facts{position:absolute;left:6rem;bottom:8rem;width:min(470px,42vw)}
.facts-label{display:block;margin-bottom:1.25rem;color:rgba(255,255,255,.48);font-size:.72rem;font-weight:600;letter-spacing:3px}
.planet-facts dl{display:grid;grid-template-columns:repeat(2,minmax(0,1fr));gap:1.1rem 2.4rem;margin:0}
.planet-facts dl div{border-top:1px solid rgba(255,255,255,.14);padding-top:.65rem}
.planet-facts dt{color:rgba(255,255,255,.42);font-size:.62rem;letter-spacing:1.5px}
.planet-facts dd{margin:.38rem 0 0;color:rgba(255,255,255,.86);font-family:'Cinzel',serif;font-size:.88rem;letter-spacing:1px}

/* ===== STARS PANEL ===== */
.stars-panel{position:relative;z-index:2;min-height:100vh;padding:7rem 4rem 5rem;display:flex;flex-direction:column;align-items:center;box-sizing:border-box}
.stars-header{max-width:700px;text-align:center;margin-bottom:2.5rem}
.stars-title{font-family:'Cinzel',serif;font-size:3rem;letter-spacing:6px;margin:0 0 .8rem}
.stars-subtitle{color:rgba(255,255,255,.7);line-height:1.8;font-size:.95rem;margin:0}
.stars-group-title{font-family:'Cinzel',serif;font-size:1.4rem;letter-spacing:4px;margin:2.2rem 0 1.4rem;color:#63b3ed;width:100%;max-width:1200px;text-align:left}
.stars-grid{display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:1.2rem;width:100%;max-width:1200px}
.star-card{position:relative;display:flex;flex-direction:column;align-items:center;text-align:center;background:rgba(255,255,255,.04);border:1px solid rgba(255,255,255,.09);border-radius:16px;padding:1.8rem 1.4rem 1.4rem;transition:transform .3s ease,border-color .3s ease,background .3s ease;overflow:hidden}
.star-card::before{content:'';position:absolute;top:0;left:0;width:100%;height:2px;background:linear-gradient(90deg,#63b3ed,transparent);opacity:0;transition:opacity .3s}
.star-card:hover{transform:translateY(-4px);border-color:rgba(99,179,237,.35);background:rgba(255,255,255,.07)}
.star-card:hover::before{opacity:1}
.star-figure{flex:0 0 auto;border-radius:50%;margin-bottom:.9rem;animation:starPulse 3s ease-in-out infinite}
.star-category{display:inline-block;font-size:.56rem;font-weight:700;letter-spacing:2px;text-transform:uppercase;padding:.28rem .7rem;border-radius:50px;margin-bottom:.5rem}
.star-category.star{color:#63b3ed;background:rgba(99,179,237,.12)}
.star-category.cluster{color:#a78bfa;background:rgba(167,139,250,.12)}
.star-name{font-family:'Cinzel',serif;font-size:1.05rem;letter-spacing:1px;margin:0 0 .25rem}
.star-constellation{font-size:.65rem;letter-spacing:2px;color:rgba(255,255,255,.5);text-transform:uppercase;margin:0 0 .6rem}
.star-description{font-size:.82rem;line-height:1.65;color:rgba(255,255,255,.72);margin:0 0 .7rem}
.star-meta{display:flex;justify-content:center;gap:.7rem;font-size:.6rem;letter-spacing:1.5px;color:rgba(255,255,255,.42);text-transform:uppercase}
@keyframes starPulse{0%,100%{transform:scale(1);filter:brightness(1)}50%{transform:scale(1.06);filter:brightness(1.18)}}

/* ===== SISTEMA PANEL ===== */
.system-panel{position:relative;z-index:2;min-height:100vh;padding:7rem 4rem 5rem;display:flex;flex-direction:column;align-items:center;box-sizing:border-box}
.system-header{max-width:700px;text-align:center;margin-bottom:2.5rem}
.system-title{font-family:'Cinzel',serif;font-size:3rem;letter-spacing:6px;margin:0 0 .8rem}
.system-subtitle{color:rgba(255,255,255,.7);line-height:1.8;font-size:.95rem;margin:0}
.system-grid{display:grid;grid-template-columns:repeat(2,minmax(0,1fr));gap:1.2rem;width:100%;max-width:900px}
.system-card{background:rgba(255,255,255,.04);border:1px solid rgba(255,255,255,.09);border-radius:16px;padding:1.7rem 1.6rem;text-align:center;transition:transform .3s ease,border-color .3s ease,background .3s ease}
.system-card:hover{transform:translateY(-4px);border-color:rgba(99,179,237,.35);background:rgba(255,255,255,.07)}
.system-value{display:block;font-family:'Cinzel',serif;font-size:1.3rem;color:#63b3ed;letter-spacing:2px;margin-bottom:.5rem}
.system-card h4{font-family:'Cinzel',serif;font-size:1rem;letter-spacing:2px;margin:0 0 .5rem}
.system-card p{font-size:.78rem;line-height:1.65;color:rgba(255,255,255,.65);margin:0}

@media(max-width:768px){
  .header{padding:1.2rem 1.5rem}
  .nav-links{gap:.5rem}
  .nav-tab{font-size:.75rem;padding:.3rem .55rem}
  .btn-enroll{display:none}
  .section{padding:0 2rem}
  .title{font-size:3.5rem}
  .detail-title{font-size:2.8rem}
  .adjacent-planet,.scroll-indicator{display:none}
  .planet-facts{left:2rem;bottom:3rem;width:calc(100% - 4rem)}
  .planet-facts dl{gap:.85rem 1rem}
  .planet-facts dd{font-size:.72rem}
  .surface-content{display:none}
  .stars-panel,.system-panel{padding:6rem 1.2rem 3rem}
  .stars-title,.system-title{font-size:2.2rem}
  .stars-grid{grid-template-columns:1fr}
  .system-grid{grid-template-columns:1fr}
  .stars-group-title{font-size:1.1rem;text-align:center}
  .nav-tab.hide-mobile{display:none}
}
</style>