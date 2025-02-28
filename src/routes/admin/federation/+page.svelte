<script lang="ts">
  import { profile } from '$lib/auth.js'
  import Placeholder from '$lib/components/ui/Placeholder.svelte'
  import EditableList from '$lib/components/ui/list/EditableList.svelte'
  import RelativeDate from '$lib/components/util/RelativeDate.svelte'
  import { publishedToDate } from '$lib/components/util/date.js'
  import { getClient } from '$lib/lemmy.js'
  import { trycatch } from '$lib/util.js'
  import type { Instance } from 'lemmy-js-client'
  import {
    Button,
    FileInput,
    Material,
    Popover,
    TextInput,
    toast,
  } from 'mono-svelte'
  import {
    Check,
    ExclamationTriangle,
    Icon,
    InformationCircle,
    Plus,
    XMark,
  } from 'svelte-hero-icons'
  import { flip } from 'svelte/animate'
  import { expoInOut, expoOut } from 'svelte/easing'
  import { slide } from 'svelte/transition'

  export let data

  let blockInstance = {
      instance: '',
      loading: false,
    },
    allowInstance = {
      instance: '',
      loading: false,
    },
    saving = false

  const moderateInstance = async (instance: string, blocked: boolean) => {
    if (instance == '') return
    await trycatch(async () => {
      if (
        !$profile?.jwt ||
        !data.federated_instances?.federated_instances?.blocked ||
        !data.federated_instances?.federated_instances?.allowed
      )
        return

      if (blocked) blockInstance.loading = true
      else allowInstance.loading = true

      const blockedInstances =
        data.federated_instances.federated_instances.blocked.map(
          (i) => i.domain
        )

      const allowedInstances =
        data.federated_instances.federated_instances.allowed.map(
          (i) => i.domain
        )

      if (blocked) blockedInstances.push(instance)
      if (!blocked) allowedInstances.push(instance)

      await getClient().editSite({
        auth: $profile.jwt,
        blocked_instances: blockedInstances,
        allowed_instances: allowedInstances,
      })

      const instances = await getClient().getFederatedInstances()

      data.federated_instances = instances

      blockInstance.instance = ''
    })

    blockInstance.loading = false
    allowInstance.loading = false
  }

  const save = async () => {
    const res = await trycatch(async () => {
      if (
        !$profile?.jwt ||
        !data.federated_instances?.federated_instances?.blocked
      )
        return

      saving = true

      const blockedInstances =
        data.federated_instances.federated_instances.blocked.map(
          (i) => i.domain
        )
      const allowedInstances =
        data.federated_instances.federated_instances.allowed.map(
          (i) => i.domain
        )

      return await getClient().editSite({
        auth: $profile.jwt,
        allowed_instances: allowedInstances,
        blocked_instances: blockedInstances,
      })
    })

    if (res) {
      toast({
        content: 'Edited your site.',
        type: 'success',
      })
    }
    saving = false
  }

  let csv: FileList | null

  async function parseCsv(files: FileList) {
    if (!(files.length > 0)) return

    const reader = new FileReader()

    reader.onload = (e) => {
      if (!data.federated_instances?.federated_instances?.blocked)
        throw new Error('Missing instance')
      const content = e.target?.result
      if (!content)
        toast({ content: 'No content in that file', type: 'warning' })

      try {
        const instances: Instance[] = []
        const str = content?.toString()
        if (!str) throw new Error('No content')

        const lines = str.split(/\r?\n/).slice(1)

        for (const line of lines) {
          if (line == '') continue
          const domain = line.split(',')[0]

          const item: Instance = {
            domain: domain,
            id: Math.floor(Math.random() * 1000000),
            published: new Date().toISOString(),
          }

          instances.push(item)
        }

        data.federated_instances.federated_instances.blocked = instances
      } catch (err) {
        toast({ content: err as 'Failed to parse CSV', type: 'error' })
      }
    }

    reader.onerror = (e) =>
      toast({ content: 'Failed to read file.', type: 'error' })

    reader.readAsText(files[0])
  }

  $: {
    if (csv) parseCsv(csv)
  }
</script>

<svelte:head>
  <title>Federation</title>
</svelte:head>

<h1 class="font-bold text-2xl flex items-center justify-between">
  Federation <Button
    color="primary"
    on:click={save}
    loading={saving}
    disabled={saving}
  >
    Save
  </Button>
</h1>
{#if data.site && data.federated_instances?.federated_instances?.blocked}
  <FileInput preview={false} bind:files={csv}>
    <Button class="w-max" slot="button">
      <Icon src={Plus} mini size="18" slot="prefix" />
      Upload a CSV
    </Button>
    <svelte:fragment slot="choose">
      {''}
    </svelte:fragment>
  </FileInput>
  <div class="flex flex-col md:flex-row gap-4">
    <div class="flex-1 w-full max-h-[42rem] h-full flex flex-col gap-2">
      <h2 class="font-bold text-lg">Blocked Instances</h2>
      <form
        on:submit|preventDefault={() =>
          moderateInstance(blockInstance.instance, true)}
        class="flex flex-row gap-2"
      >
        <TextInput
          placeholder="Block an instance..."
          class="flex-1"
          bind:value={blockInstance.instance}
        />
        <Button
          submit
          loading={blockInstance.loading}
          disabled={blockInstance.loading}
        >
          Block
        </Button>
      </form>
      <Material class="h-full overflow-auto" color="distinct">
        <EditableList
          class="[&>*]:py-3 dark:!divide-zinc-800 {allowInstance.instance != ''
            ? 'opacity-50'
            : ''}"
        >
          {#if data.federated_instances.federated_instances.blocked.length > 0}
            {#each data.federated_instances.federated_instances.blocked.sort( (b, a) => b.domain.localeCompare(a.domain) ) as instance (instance.id)}
              <div
                animate:flip={{ duration: 300, easing: expoOut }}
                class="flex justify-between items-center first:pt-0 last:pb-0"
              >
                <div class="flex flex-col">
                  <span class="font-medium">{instance.domain}</span>
                  <span
                    class="text-xs text-slate-600 dark:text-zinc-400 capitalize"
                  >
                    {instance.software ?? 'Unknown'} • <RelativeDate
                      date={publishedToDate(instance.published)}
                    />
                  </span>
                </div>
                <Button
                  size="square-md"
                  on:click={() => {
                    if (!data.federated_instances?.federated_instances?.blocked)
                      return

                    data.federated_instances.federated_instances.blocked =
                      data.federated_instances?.federated_instances?.blocked.filter(
                        (i) => i.id != instance.id
                      )
                  }}
                >
                  <Icon src={XMark} size="16" mini slot="prefix" />
                </Button>
              </div>
            {/each}
          {:else}
            <Placeholder
              icon={Check}
              title="No blocked instances"
              description="Blocking an instance will hide submissions from that instance."
            />
          {/if}
        </EditableList>
      </Material>
    </div>
    <div class="md:flex-1 w-full max-h-[42rem] flex flex-col gap-2">
      <h2 class="font-bold text-lg flex items-center space-x-1">
        <span>Allowed Instances</span>
        {#if allowInstance.instance || !(data.federated_instances.federated_instances.allowed?.length == 0)}
          <Popover openOnHover origin="bottom-right">
            <Icon
              src={ExclamationTriangle}
              solid
              class="text-yellow-500"
              slot="target"
              size="20"
            />
            <p class="font-normal">
              Adding items to "Allowed Instances" will <strong>
                block every other instance.
              </strong>
            </p>
          </Popover>
        {/if}
      </h2>
      <form
        on:submit|preventDefault={() =>
          moderateInstance(allowInstance.instance, false)}
        class="flex flex-row gap-2"
      >
        <TextInput
          placeholder="Allow an instance..."
          class="flex-1"
          bind:value={allowInstance.instance}
        />
        <Button
          submit
          loading={allowInstance.loading}
          disabled={allowInstance.loading}
        >
          Allow
        </Button>
      </form>
      <Material class="h-full overflow-auto" color="distinct">
        <EditableList class="[&>*]:py-3 dark:!divide-zinc-800 ">
          {#if data.federated_instances.federated_instances.allowed.length > 0}
            {#each data.federated_instances.federated_instances.allowed.sort( (b, a) => b.domain.localeCompare(a.domain) ) as instance (instance.id)}
              <div
                animate:flip={{ duration: 300, easing: expoOut }}
                class="flex justify-between items-center first:pt-0 last:pb-0"
              >
                <div class="flex flex-col">
                  <span class="font-medium">{instance.domain}</span>
                  <span
                    class="text-xs text-slate-600 dark:text-zinc-400 capitalize"
                  >
                    {instance.software ?? 'Unknown'} • <RelativeDate
                      date={publishedToDate(instance.published)}
                    />
                  </span>
                </div>
                <Button
                  size="square-md"
                  on:click={() => {
                    if (!data.federated_instances?.federated_instances?.blocked)
                      return

                    data.federated_instances.federated_instances.blocked =
                      data.federated_instances?.federated_instances?.blocked.filter(
                        (i) => i.id != instance.id
                      )
                  }}
                >
                  <Icon src={XMark} size="16" mini slot="prefix" />
                </Button>
              </div>
            {/each}
          {:else}
            <div class="my-auto h-max">
              <Placeholder
                icon={Check}
                title="No allowed instances"
                description="Adding an item to this will block every other instance."
              />
            </div>
          {/if}
        </EditableList>
      </Material>
    </div>
  </div>
{/if}
