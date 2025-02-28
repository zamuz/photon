<script lang="ts">
  import Markdown from '$lib/components/markdown/Markdown.svelte'
  import Avatar from '$lib/components/ui/Avatar.svelte'
  import StickyCard from '$lib/components/ui/StickyCard.svelte'
  import FormattedNumber from '$lib/components/util/FormattedNumber.svelte'
  import RelativeDate from '$lib/components/util/RelativeDate.svelte'
  import { publishedToDate } from '$lib/components/util/date.js'
  import { getClient } from '$lib/lemmy.js'
  import type { PersonView, SiteView, Tagline } from 'lemmy-js-client'
  import { Button, Popover } from 'mono-svelte'
  import {
    BuildingOffice,
    Calendar,
    ChatBubbleOvalLeftEllipsis,
    Icon,
    Newspaper,
    PencilSquare,
    ServerStack,
    UserGroup,
  } from 'svelte-hero-icons'

  export let site: SiteView
  export let taglines: Tagline[] | undefined = undefined
  export let admins: PersonView[] | undefined = undefined
  export let version: string | undefined = undefined
</script>

<StickyCard>
  <div class="flex flex-row gap-3 items-center">
    {#if site.site.icon}
      <Avatar width={42} url={site.site.icon} alt={site.site.name} />
    {/if}
    <div class="flex flex-col">
      <h1 class="font-bold text-base">{site.site.name}</h1>
      <span class="text-sm opacity-60">
        {new URL(site.site.actor_id).hostname}
      </span>
    </div>
  </div>
  <span class="flex flex-row items-center gap-1 text-sm" title="Created">
    <Icon src={Calendar} width={16} height={16} mini />
    <RelativeDate date={publishedToDate(site.site.published)} />
  </span>
  <div class="text-sm flex flex-row flex-wrap gap-3" title="Users">
    <span class="flex flex-row items-center gap-1">
      <Icon src={UserGroup} width={16} height={16} mini />
      <FormattedNumber number={site.counts.users} />
    </span>
    <span class="flex flex-row items-center gap-1" title="Posts">
      <Icon src={PencilSquare} width={16} height={16} mini />
      <FormattedNumber number={site.counts.posts} />
    </span>
    <span class="flex flex-row items-center gap-1" title="Comments">
      <Icon src={ChatBubbleOvalLeftEllipsis} width={16} height={16} mini />
      <FormattedNumber number={site.counts.comments} />
    </span>
    <span class="flex flex-row items-center gap-1" title="Communities">
      <Icon src={Newspaper} width={16} height={16} mini />
      <FormattedNumber number={site.counts.communities} />
    </span>
  </div>
  <div class="flex flex-row items-center">
    <Button href="/modlog" size="md" class="rounded-r-none">
      <Icon src={Newspaper} size="16" mini />
      Modlog
    </Button>
    <Button href="/legal" size="md" class="rounded-l-none border-l-0">
      <Icon src={BuildingOffice} size="16" mini />
      Legal
    </Button>
  </div>

  {#if admins}
    <span class="font-bold">Admins</span>
    <div
      class="flex items-center -space-x-1 flex-wrap hover:space-x-1 transition-all
      cursor-pointer"
    >
      {#each admins as admin}
        <Popover openOnHover origin="top-left" class="transition-all">
          <a
            class="block ring rounded-full ring-slate-50 dark:ring-zinc-950 transition-all"
            href="/u/{admin.person.name}@{new URL(admin.person.actor_id)
              .hostname}"
            slot="target"
          >
            <Avatar
              width={28}
              url={admin.person.avatar}
              alt={admin.person.name}
            />
          </a>
          <span class="font-bold">{admin.person.name}</span>
        </Popover>
      {/each}
    </div>
  {/if}
  <hr class="border-slate-300 dark:border-zinc-700" />
  {#if taglines && taglines.length > 0}
    <Markdown
      source={taglines[Math.floor(Math.random() * taglines.length)].content}
    />
    <hr class="border-slate-300 dark:border-zinc-700" />
  {/if}
  <Markdown source={site.site.description} />
  <hr class="border-slate-300 dark:border-zinc-700" />
  <Markdown source={site.site.sidebar} />
  {#if version}
    <hr class="border-slate-300 dark:border-zinc-700" />
    <span
      class="text-slate-600 flex flex-row items-center gap-1 dark:text-zinc-400"
      title="Lemmy version"
    >
      <Icon src={ServerStack} mini size="16" />
      {version}
    </span>
  {/if}
</StickyCard>
