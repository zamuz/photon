<script lang="ts">
  import Comment from '$lib/components/lemmy/comment/Comment.svelte'
  import Post from '$lib/components/lemmy/post/Post.svelte'
  import { Select, toast } from 'mono-svelte'
  import { getClient } from '$lib/lemmy.js'
  import { isCommentView, isPostView } from '$lib/lemmy/item.js'
  import type { CommentView, PostView } from 'lemmy-js-client'
  import { profile } from '$lib/auth.js'
  import MarkdownEditor from '$lib/components/markdown/MarkdownEditor.svelte'
  import { Fire, Icon, Trash } from 'svelte-hero-icons'
  import MultiSelect from '$lib/components/input/Switch.svelte'
  import { removalTemplate } from '$lib/components/lemmy/moderation/moderation.js'
  import { userSettings } from '$lib/settings.js'
  import { fullCommunityName } from '$lib/util.js'
  import { amMod, isAdmin } from './moderation'
  import { Button, Checkbox, Modal } from 'mono-svelte'

  export let open: boolean
  export let item: PostView | CommentView | undefined = undefined
  export let purge: boolean = false

  let reason = ''
  let commentReason: boolean = false
  let privateMessage: boolean = false
  let loading = false
  let preset = $userSettings.moderation.presets[0]?.content ?? ''

  $: removed = item
    ? isCommentView(item)
      ? item.comment.removed
      : item.post.removed
    : false

  const getReplyReason = (reason: string, preset: string) => {
    if (!item) return `no template`

    return removalTemplate(preset, {
      communityLink: `!${fullCommunityName(
        item!.community.name,
        item!.community.actor_id
      )}`,
      postTitle: item.post.name,
      reason: reason,
      username: item.creator.name,
    })
  }

  $: replyReason = commentReason ? getReplyReason(reason, preset) : ''

  async function remove() {
    if (!item) return
    if (!$profile?.jwt) throw new Error('Unauthenticated')

    loading = true

    try {
      if (purge) {
        if (isCommentView(item)) {
          await getClient(undefined, fetch).purgeComment({
            auth: $profile.jwt,
            comment_id: item.comment.id,
            reason: reason,
          })
        } else {
          await getClient(undefined, fetch).purgePost({
            auth: $profile.jwt,
            post_id: item.post.id,
            reason: reason,
          })
        }

        toast({
          content: 'Successfully purged that submission.',
          type: 'success',
        })

        loading = false
        open = false

        return
      }

      if (commentReason) {
        if (replyReason == '') {
          toast({
            content: 'Your reply cannot be empty if "Reply reason" is enabled.',
          })
          return
        }

        if (privateMessage) {
          await getClient()
            .createPrivateMessage({
              auth: $profile.jwt,
              content: replyReason,
              recipient_id: isCommentView(item)
                ? item.comment.creator_id
                : item.post.creator_id,
            })
            .catch(() => {
              toast({
                content: 'Failed to message user. Removing anyway...',
                type: 'warning',
              })
            })
        } else {
          await getClient()
            .createComment({
              auth: $profile.jwt,
              content: replyReason,
              post_id: item.post.id,
              parent_id: isCommentView(item) ? item.comment.id : undefined,
            })
            .catch(() => {
              toast({
                content: 'Failed to post reply. Removing anyway...',
                type: 'warning',
              })
            })
        }
      }

      if (isCommentView(item)) {
        await getClient().removeComment({
          auth: $profile.jwt,
          comment_id: item.comment.id,
          removed: !removed,
          reason: reason || undefined,
        })
        item.comment.removed = !removed
      } else if (isPostView(item)) {
        await getClient().removePost({
          auth: $profile.jwt,
          post_id: item.post.id,
          removed: !removed,
          reason: reason || undefined,
        })

        item.post.removed = !removed
      }

      open = false

      toast({
        content: `Successfully ${
          removed ? 'restored' : 'removed'
        } that submission.`,
        type: 'success',
      })
    } catch (err) {
      toast({
        content: err as any,
        type: 'error',
      })
    }

    loading = false
  }

  const resetText = () => {
    reason = ''
    replyReason = ''
    commentReason = false
  }

  $: {
    if (item) {
      resetText()
    }
  }
</script>

<Modal bind:open>
  <span slot="title">
    {purge ? 'Purging' : removed ? 'Restoring' : 'Removing'} submission
  </span>
  {#if item}
    <form
      on:submit|preventDefault={remove}
      class="flex flex-col gap-4 list-none"
    >
      {#if isCommentView(item)}
        <Comment
          node={{
            children: [],
            comment_view: item,
            depth: 1,
          }}
          postId={item.post.id}
          actions={false}
        />
      {:else if isPostView(item)}
        <Post actions={false} post={item} />
      {/if}

      <MarkdownEditor
        rows={3}
        label="Reason"
        placeholder="Optional"
        bind:value={reason}
      />

      {#if !removed && $profile?.user && (amMod($profile.user, item.community) || (isAdmin($profile.user) && item.community.local))}
        <Checkbox bind:checked={commentReason}>Reply with reason</Checkbox>

        {#if commentReason}
          <MultiSelect
            options={[false, true]}
            optionNames={['Comment', 'Message']}
            bind:selected={privateMessage}
          />
          <MarkdownEditor
            bind:value={replyReason}
            placeholder={replyReason}
            rows={3}
          >
            <div class="flex justify-between items-end mb-1" slot="label">
              Reply
              <Select bind:value={preset} placeholder="No preset">
                {#each $userSettings.moderation.presets as preset}
                  <option value={preset.content}>
                    {preset.title}
                  </option>
                {/each}
              </Select>
            </div>
          </MarkdownEditor>
        {/if}
      {/if}

      <Button
        color={purge ? 'danger' : 'primary'}
        size="lg"
        {loading}
        disabled={loading}
        submit
      >
        <Icon src={purge ? Fire : Trash} mini size="16" slot="prefix" />
        {#if purge}
          Purge
        {:else}
          {removed ? 'Restore' : 'Remove'}
        {/if}
      </Button>
    </form>
  {/if}
</Modal>
