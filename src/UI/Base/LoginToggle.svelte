<script lang="ts">
  import Loading from "./Loading.svelte"
  import type { OsmServiceState } from "../../Logic/Osm/OsmConnection"
  import { OsmConnection } from "../../Logic/Osm/OsmConnection"
  import { Translation } from "../i18n/Translation"
  import Translations from "../i18n/Translations"
  import Tr from "./Tr.svelte"
  import { ImmutableStore, UIEventSource } from "../../Logic/UIEventSource"
  import Invalid from "../../assets/svg/Invalid.svelte"
  import ArrowPath from "@babeard/svelte-heroicons/mini/ArrowPath"

  export let state: {
    osmConnection: OsmConnection
    featureSwitches?: { featureSwitchEnableLogin?: UIEventSource<boolean> }
  }
  /**
   * If set, 'loading' will act as if we are already logged in.
   */
  export let ignoreLoading: boolean = false
  /**
   * Only show the 'successful' state, don't show loading or error messages
   */
  export let silentFail: boolean = false
  let loadingStatus = state?.osmConnection?.loadingStatus ?? new ImmutableStore("logged-in")
  let badge = state?.featureSwitches?.featureSwitchEnableLogin ?? new ImmutableStore(true)
  const t = Translations.t.general
  const offlineModes: Partial<Record<OsmServiceState, Translation>> = {
    offline: t.loginFailedOfflineMode,
    unreachable: t.loginFailedUnreachableMode,
    unknown: t.loginFailedUnreachableMode,
    readonly: t.loginFailedReadonlyMode,
  }
  const apiState =
    state?.osmConnection?.apiIsOnline ?? new ImmutableStore<OsmServiceState>("online")
</script>

{#if $badge}
  {#if !ignoreLoading && !silentFail && $loadingStatus === "loading"}
    <slot name="loading">
      <Loading />
    </slot>
  {:else if !silentFail && $loadingStatus === "error"}
    <slot name="error">
      <div class="alert flex flex-col items-center">
        <div class="max-w-64 flex items-center">
          <Invalid class="m-2 h-8 w-8 shrink-0" />
          <Tr t={offlineModes[$apiState] ?? t.loginFailedUnreachableMode} />
        </div>
        <button class="h-fit" on:click={() => state.osmConnection.AttemptLogin()}>
          <ArrowPath class="h-6 w-6" />
          <Tr t={t.retry} />
        </button>
      </div>
    </slot>
  {:else if $loadingStatus === "logged-in"}
    <slot />
  {:else if !silentFail && $loadingStatus === "not-attempted"}
    <slot name="not-logged-in" />
  {/if}
{/if}
