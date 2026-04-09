<template>
  <div class="hero-wrapper">
    <Carousel
      :items="slides"
      variant="hero"
      :modal="true"
      :autoplay="true"
      :autoplay-delay="5000"
      :loop="true"
      hero-height="80vh"
      cta-label="View More"
      @modal-open="onModalOpen"
      @cta-click="onCtaClick"
    >
      <template #hero-action="{ item }">
        <button class="hero-extra-btn" @click="onSave(item)">Save</button>
      </template>
    </Carousel>
  </div>
</template>

<script>
import Carousel from "@/components/Carousel.vue"

export default {
  name: 'Hero',
  components: { Carousel },
  data() {
    return {
      slides: [
        {
          title: 'Precision. Printed. Delivered.',
          description: 'Every product is designed, 3D printed and hand-assembled in our studio — made to last and built to impress.',
          fullText: 'We combine industrial-grade FDM and resin printing with careful hand assembly. Each piece goes through a quality check before shipping, so what arrives at your door is exactly what you envisioned — precise, durable and uniquely yours.',
          tag: 'Our Process',
          emoji: '🖨️',
          color: '#0f172a',
        },
        {
          title: 'Custom Parts, Zero Compromises',
          description: 'Send us your design or describe what you need — we\'ll handle everything from prototyping to the final product.',
          fullText: 'Whether you need a one-off prototype or a small production run, we work directly from your STL files or help you design from scratch. Tolerances, materials, colors and finishes are all fully configurable to match your specifications.',
          tag: 'Custom Orders',
          emoji: '⚙️',
          color: '#1e1b4b',
        },
        {
          title: 'Home Décor Reimagined',
          description: 'Geometric vases, modular shelves, sculptural lamp shades — objects that spark conversation in any room.',
          fullText: 'Our home collection blends parametric design with everyday functionality. Each piece is printed in high-detail PLA+ or PETG and assembled with concealed hardware, giving a clean finish that looks anything but printed.',
          tag: 'Home Collection',
          emoji: '🏠',
          color: '#292524',
        },
        {
          title: 'Gadgets & Desk Accessories',
          description: 'Cable managers, phone stands, keyboard wrist rests — small things that make your workspace work better.',
          fullText: 'Designed for real desks and real workflows, our accessories are ergonomic, stackable and available in a range of colors. Each component snaps together with precision-fit joints, no glue or screws required.',
          tag: 'Workspace',
          emoji: '🖥️',
          color: '#14532d',
        },
        {
          title: 'Small Batches. Big Quality.',
          description: 'We don\'t mass produce. Every order is printed fresh and assembled by hand in limited quantities.',
          fullText: 'Small-batch production means we can maintain strict quality standards and experiment with new materials and designs more freely. It also means every item you receive is produced specifically for your order — never sitting in a warehouse.',
          tag: 'Philosophy',
          emoji: '✦',
          color: '#3b1f0f',
        },
      ],
    }
  },
  methods: {
    onModalOpen(item) {
      console.log('Modal opened:', item.title)
    },
    onCtaClick(item) {
      console.log('CTA clicked:', item.title)
    },
    onSave(item) {
      console.log('Saved:', item.title)
    },
  },
}
</script>

<style>
.hero-wrapper {
  width: 100vw;
  height: 70vh;
  background: var(--background);
  position: relative;
  z-index: 1;
}

.hero-extra-btn {
  background: rgba(255, 255, 255, 0.15);
  color: #fff;
  border: 1px solid rgba(255, 255, 255, 0.35);
  padding: 11px 22px;
  border-radius: 99px;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  backdrop-filter: blur(4px);
  transition: background 0.2s;
}
.hero-extra-btn:hover {
  background: rgba(255, 255, 255, 0.28);
}
</style>