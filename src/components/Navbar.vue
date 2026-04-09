<script>
/*
  Copyright - 2026 joe-watson-sbf (Joseph Watzson) 

  Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), 
  to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense,
  and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

  The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
  WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
  Modified by Nfire07
*/
import ThemeChanger from './ThemeChanger.vue';
import LanguageChanger from './LanguageChanger.vue';
export default {
  name: 'Navbar',
  props: {
    links: {
      type: Array,
      default: () => [],
    },
    iconLabel: {
      type: String,
      default: 'Home'
    },
    iconPath:{
      type: String,
      default: "/favicon.ico"
    }
  },
  computed: {
    computedLinks() {
      return ["icon", ...this.links];
    }
  },
  components:{
    ThemeChanger,
    LanguageChanger,
  }
};
</script>

<style scoped>
.navbar-wrapper {
  display: flex;
  align-items: center;
  justify-content: flex-start;
  width: 100%;
  padding: 1rem;
  box-sizing: border-box;
  position:relative;
  position: absolute;
  top: 0;
  left: 0;
  z-index: 100;
  background-color: transparent;
}

.card {
  height: 120px;
  border-radius: 12px;
  display: flex;
  gap: 8px;
  padding: 0.5rem;
  width: auto; 
  max-width: 100%; 
  overflow-x: auto; 
}

.card::-webkit-scrollbar { display: none; }
.card { -ms-overflow-style: none; scrollbar-width: none; }

.icon-slot, .link-slot {
  height: 100%;
  flex: 0 0 auto;
  overflow: hidden;
  border-radius: 8px;
  transition: max-width 0.4s ease-in-out, width 0.4s ease-in-out, background 0.4s;
}

.link-slot{
  box-shadow: rgba(0, 0, 0, 0.4) 0px 2px 4px, rgba(0, 0, 0, 0.3) 0px 7px 13px -3px, rgba(0, 0, 0, 0.2) 0px -3px 0px inset;
}


.icon-slot {
  width: max-content; 
  max-width: 60px; 
  border: none;
}

.icon-slot:hover {
  max-width: 500px; 
}

.icon-link {
  display: flex;
  align-items: center;
  gap: 12px;
  text-decoration: none;
  width: max-content;
  height: 100%;
  padding: 0 10px;
  box-sizing: border-box;
}

.icon-img {
  width: 40px;
  height: 40px;
  object-fit: contain;
  flex-shrink: 0;
}

.icon-label {
  opacity: 0;
  white-space: nowrap; 
  font-size: 12px;
  font-weight: 400;
  color: var(--foreground);
  text-shadow: 2px 2px #020202;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  transition: opacity 0.3s ease 0.15s;
}

.icon-slot:hover .icon-label {
  opacity: 1;
}

.link-slot {
  width: 40px; 
}

.link-slot:hover {
  width: 120px; 
}

.nav-link {
  display: flex;
  justify-content: center;
  align-items: center;
  text-decoration: none;
  width: 100%;
  height: 100%;
}

.nav-link span {
  white-space: nowrap;
  transform: rotate(-90deg);
  transition: transform 0.4s ease-in-out;
  text-transform: uppercase;
  font-weight: 800;
  color: hsl(from var(--foreground) h s calc(l * 1.5)); 
  letter-spacing: 0.1em;
  text-shadow: 2px 2px #020202;
  font-size: 14px;
}

.link-slot:hover .nav-link span {
  transform: rotate(0deg);
}

@media (max-width: 600px) {
  .navbar-wrapper {
    padding: 0.5rem;
  }
  
  .card {
    height: 70px;
    gap: 5px;
  }

  .icon-slot { 
    max-width: 50px; 
  }
  .icon-slot:hover { 
    max-width: 350px; 
  }
  .icon-img { width: 30px; height: 30px; }
  
  .link-slot { width: 32px; }
  .link-slot:hover { width: 100px; }
  
  .nav-link span { font-size: 11px; }
}
</style>

<template>
  <nav class="navbar-wrapper">
    <div class="card">
      <div
        v-for="(link, index) in computedLinks"
        :key="index"
        :class="link === 'icon' ? 'icon-slot' : 'link-slot'"
      >
        <template v-if="link === 'icon'">
          <a href="/home" class="icon-link">
            <img :src="iconPath" alt="APPLICATION ICON" class="icon-img"/>
            <span class="icon-label" v-html="iconLabel"></span>
          </a>
        </template>

        <template v-else>
          <a :href="`/${link.toLowerCase()}`" class="nav-link">
            <span>{{ link }}</span>
          </a>
        </template>
      </div>
    </div>
    <ThemeChanger/>
    <LanguageChanger/>
  </nav>
</template>