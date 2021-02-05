<script>
  import { isActive, url, layout } from "@roxi/routify"; // import Login from './login/index.svelte'
  import { metatags } from "@roxi/routify";
  // metatags.title = 'Nebula星云图分析'
  metatags.description = "以开源的形式，做最好的图分析";
</script>

index
{#each $layout.children as { path, title }}
  <li class:active={$isActive(path)}>
    <a href={$url(path)}>{title}</a>
  </li>
{/each}
