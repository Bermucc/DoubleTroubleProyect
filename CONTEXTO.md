# Double Trouble Company — Contexto del proyecto

## Resumen
Web de minijuegos para beber con amigos en fiestas. Sin base de datos, datos guardados en localStorage.

## Stack técnico
- **Framework**: Astro 7
- **Estilos**: Tailwind CSS v4 (configurado vía CSS con `@theme`)
- **Animaciones**: GSAP
- **Audio**: Web Audio API (sintetizado, sin archivos externos)
- **Package manager**: pnpm
- **Hosting**: Vercel (gratuito)
- **Repo**: https://github.com/Bermucc/DoubleTroubleProyect

## Estructura de archivos
```
double-trouble/
├── public/
│   └── favicon.svg
├── src/
│   ├── styles/global.css          # Tailwind + colores tema (verdosos)
│   ├── layouts/Layout.astro       # Layout principal (header, footer, SEO, Google Fonts)
│   ├── components/
│   │   ├── Header.astro           # Navbar fijo con logo
│   │   ├── Footer.astro           # Footer con copyright
│   │   └── GameCard.astro         # Card reutilizable para minijuegos
│   └── pages/
│       ├── index.astro            # Landing page con GSAP animations
│       ├── games.astro            # Hub de juegos
│       └── games/
│           ├── traga-chupitos.astro   # Máquina tragaperras
│           └── ruleta-vodka.astro     # Ruleta de casino
├── astro.config.mjs
├── package.json
└── pnpm-lock.yaml
```

## Paleta de colores (verdosos)
```css
--color-dt-green: #22C55E;
--color-dt-green-light: #6EE7B7;
--color-dt-emerald: #10B981;
--color-dt-dark: #0A0A0F;
--color-dt-darker: #06060B;
--color-dt-light: #F5F3FF;
```

## Páginas

### `/` — Landing
- Hero con texto "Double Trouble Company" en gradiente verde
- GSAP animaciones de entrada
- Sección "¿Cómo funciona?" con 3 pasos
- CTA hacia /games

### `/games` — Hub de juegos
Cards con GameCard.astro. Juegos disponibles:
1. **Tragachupitos** (`/games/traga-chupitos`)
2. **Ruleta Vodka** (`/games/ruleta-vodka`)

### `/games/traga-chupitos` — Máquina tragaperras
- Cuerpo de máquina con bordes cromados y tornillos
- 3 carretes que muestran tiras de símbolos (`🍒🍋🍊🍇🍉🥃`)
- Los carretes giran hacia abajo (desplazamiento vertical con GSAP)
- Los símbolos se desplazan por los carretes sin blur (visibles nítidos)
- Palanca roja (con forma de bola y eje metálico)
- Botones glass (efecto translúcido)
- Línea de pago con punto luminoso verde

#### Mecánica del juego:
- **Giro continuo**: los 3 carretes siempre están girando en bucle
- Al tirar la palanca: chispas + sacudida + se determinan 3 resultados
- Resultados se revelan uno por uno (staggered stops con `power4.out`)
- **3 🥃** → ¡BEBE UN CHUPITO! + confeti verde + jingle
- **2 🥃 en los primeros 2 carretes** → flujo dramático:
  - Zoom sobre los 3 carretes (clase CSS `dramatic-glow`)
  - Overlay con texto (¡JACKPOT! si 3 🥃, ¡A punto! si no)
  - Revelación lenta de cada carrete (`expo.out`)
  - Redoble de tambores en el 3er carrete
  - Si el 3º no es 🥃 → ¡FIASCO! en rojo
  - Si el 3º es 🥃 → ¡BEBE! en verde
- **Sistema de calentón**: tiradas consecutivas sin ningún 🥃 acumulan 🔥x1-x9
  - Barra naranja/roja visible
  - A mayor calentón: carretes más rápidos (`speed = max(0.35, 1 - buildup*0.065)`)
  - Probabilidad de 🥃 aumenta (`weight = 1 + buildup*0.6`)
  - Si llega a 10 → 💀 ¡CHUPITAZO! (5 bebidas extra, resetea calentón)
  - 3 🥃 → resetea calentón a 0

#### Sonidos (Web Audio API):
- `tick()`: clic metálico al girar carretes
- `stopClunk()`: golpe grave al parar carrete
- `reelBuzz()`: zumbido durante giro
- `leverClick()`: clic al tirar palanca
- `leverSpring()`: sonido de muelle al soltar
- `drumroll()`: redoble de tambores (ruido blanco filtrado)
- `tension()`: notas tensas ascendentes
- `win()`: jingle ascendente (C-E-G-C)
- `lose()`: notas descendentes
- `fail()`: sonido grave de decepción

#### Animaciones:
- Palanca: `y: 38` bajada, luego `bounce.out` al subir
- Carretes normales: `power4.out`
- Revelación dramática: `expo.out` (más lento al final)
- Zoom: `scale: 1.18` con `power2.inOut`
- Chispas: partículas doradas con `power2.out`
- Confeti: piezas de colores cayendo
- Overlay tensión: pulso con `tensionPulse` keyframe

#### Funciones clave JS:
```javascript
buildStrip(el, resultSymbol)     // Construye tira de símbolos con resultado en targetIndex
animateReelDown(el, result, dur) // Anima tira hacia abajo
killContinuous()                 // Mata tweens de giro continuo
startContinuous()                // Inicia giro continuo
spin()                           // Función principal (async)
fireSparks(count)                // Partículas desde la palanca
fireConfetti()                   // Confeti de celebración
showTensionOverlay(text)         // Overlay de tensión
getWeightedSymbol()              // Símbolo con probabilidad sesgada por calentón
```

### `/games/ruleta-vodka` — Ruleta de casino
- Ruleta europea con 37 números (0-36) en orden real
- `conic-gradient` para los sectores de colores
- Números posicionados radialmente con `Math.cos/sin`
- Flecha dorada fija en la parte superior
- Bola central decorativa
- Botones glass de apuesta: Rojo / Negro / Verde
- Historial de últimos 12 resultados (bolitas de colores)

#### Mecánica:
- Jugador elige color (rojo/negro/verde) y pulsa GIRAR
- Ruleta gira 7-10 vueltas completas con `power4.out`
- Sonido de tick que se ralentiza con la ruleta
- Flecha rebota al parar (`yoyo: true, repeat: 5`)
- **Apuesta rojo/negro**: si acierta → BEBES, si no → te salvas
- **0 verde (sin apostar verde)**: todos beben 2
- **Apostar verde y acierta (0)**: JACKPOT, los demás beben 3

#### Fórmula de rotación:
```javascript
const sectorCenter = 360 - (resultIndex + 0.5) * SLICE;
const fullSpins = 360 * (7 + Math.floor(Math.random() * 4));
const totalDeg = fullSpins + sectorCenter;
// Usando rotation: "+=" con GSAP para rotación absoluta acumulativa
```

## Comandos
```bash
cd double-trouble
pnpm dev        # Desarrollo
pnpm build      # Build producción
pnpm preview    # Previsualizar build
```

## Notas importantes
- No usar `git commit --amend` ni `--no-verify`
- No commitear archivos con secretos/tokens
- Tailwind v4: configuración vía CSS con `@theme` (sin tailwind.config.js)
- GSAP: importado en el `<script>` de cada página
- Las páginas de juegos usan `<style is:global>` para estilos específicos
- El token de GitHub fue eliminado del remote URL por seguridad
