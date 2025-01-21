// components/PomegranateParticles.vue
<template>
    <div class="relative">
        <canvas ref="canvas" width="1080" height="150" class="w-full" @mousemove="handleMouseMove"
            @mouseleave="handleMouseLeave"></canvas>
    </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'

const canvas = ref(null)
let ctx = null
let particles = []
let animationFrameId = null
let mousePos = { x: -1000, y: -1000 }
let arilImage = null

class Particle {
    constructor(x, y) {
        this.x = x
        this.y = y
        this.originalX = x
        this.originalY = y
        this.vx = 0
        this.vy = 0
        this.rotation = Math.random() * Math.PI * 2
        this.size = 25 + Math.random() * 8 // Tamaños entre 25 y 33px
    }

    draw() {
        if (!arilImage) return

        ctx.save()
        ctx.translate(this.x, this.y)
        ctx.rotate(this.rotation)
        ctx.drawImage(arilImage, -this.size / 2, -this.size / 2, this.size, this.size)
        ctx.restore()
    }

    update() {
        const dx = mousePos.x - this.x
        const dy = mousePos.y - this.y
        const distance = Math.sqrt(dx * dx + dy * dy)

        if (distance < 60) { // Aumentado para mejor interacción con arilos más grandes
            const force = (60 - distance) / 60
            this.vx -= (dx / distance) * force * 2
            this.vy -= (dy / distance) * force * 2
        }

        this.vx += (this.originalX - this.x) * 0.05
        this.vy += (this.originalY - this.y) * 0.05

        this.vx *= 0.95
        this.vy *= 0.95

        this.x += this.vx
        this.y += this.vy
    }
}

const init = async () => {
    const canvasEl = canvas.value
    ctx = canvasEl.getContext('2d')

    // Cargar imagen
    arilImage = new Image()
    arilImage.src = '/images/aril.png'
    await new Promise(resolve => arilImage.onload = resolve)

    // Crear múltiples líneas de partículas
    const NUM_LINES = 3
    const PARTICLES_PER_LINE = 80 // Reducido para compensar el mayor tamaño
    const centerY = 75
    const lineSpacing = 25 // Aumentado para dar más espacio entre líneas

    for (let line = 0; line < NUM_LINES; line++) {
        const lineOffset = (line - (NUM_LINES - 1) / 2) * lineSpacing

        for (let i = 0; i < PARTICLES_PER_LINE; i++) {
            const progress = i / PARTICLES_PER_LINE
            const x = 40 + (1000 * progress)

            const baseY = centerY + lineOffset
            const waveHeight = 15
            const frequency = 2 * Math.PI
            const phaseShift = line * (Math.PI / NUM_LINES)
            const y = baseY + Math.sin(progress * frequency + phaseShift) * waveHeight

            const randomOffset = (Math.random() - 0.5) * 8

            particles.push(new Particle(
                x + (Math.random() - 0.5) * 15,
                y + randomOffset
            ))
        }
    }

    animate()
}

const animate = () => {
    ctx.clearRect(0, 0, 1080, 150)
    particles.forEach(particle => {
        particle.update()
        particle.draw()
    })
    animationFrameId = requestAnimationFrame(animate)
}

const handleMouseMove = (e) => {
    const rect = canvas.value.getBoundingClientRect()
    const scaleX = 1080 / rect.width
    mousePos.x = (e.clientX - rect.left) * scaleX
    mousePos.y = (e.clientY - rect.top) * scaleX
}

const handleMouseLeave = () => {
    mousePos.x = -1000
    mousePos.y = -1000
}

onMounted(() => {
    init()
})

onBeforeUnmount(() => {
    if (animationFrameId) {
        cancelAnimationFrame(animationFrameId)
    }
})
</script>