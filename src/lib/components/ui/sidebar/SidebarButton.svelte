<script lang="ts">
  import { Button } from 'mono-svelte'

  export let expanded: boolean
</script>

<Button
  {...$$restProps}
  color="tertiary"
  alignment="left"
  on:click
  class="hover:bg-slate-100 font-normal {expanded
    ? ''
    : '!p-1.5'} {$$props.class}"
>
  <slot />
</Button>
