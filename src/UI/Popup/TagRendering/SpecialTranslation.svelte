<script lang="ts">
  import { Translation } from "../../i18n/Translation"
  import SpecialVisualizations from "../../SpecialVisualizations"
  import Locale from "../../i18n/Locale"
  import type {
    RenderingSpecification,
    SpecialVisualizationState,
  } from "../../SpecialVisualization"
  import { Utils } from "../../../Utils.js"
  import type { Feature } from "geojson"
  import { UIEventSource } from "../../../Logic/UIEventSource.js"
  import ToSvelte from "../../Base/ToSvelte.svelte"
  import LayerConfig from "../../../Models/ThemeConfig/LayerConfig"
  import WeblateLink from "../../Base/WeblateLink.svelte"
  import BaseUIElement from "../../BaseUIElement"

  /**
   * The 'specialTranslation' renders a `Translation`-object, but interprets the special values as well
   */
  export let t: Translation
  export let state: SpecialVisualizationState
  export let tags: UIEventSource<Record<string, string>>
  export let feature: Feature
  export let layer: LayerConfig | undefined

  export let clss: string = ""

  let language = Locale.language
  let lang = t?.actualLanguage($language)
  let txt: string = t?.textFor($language)
  let specs: RenderingSpecification[] = []
  $: {
    try {
      txt = t.textFor($language)
      lang = t.actualLanguage($language)
      if (txt !== undefined) {
        let key = "cached_special_spec_" + $language
        specs = t[key]
        if (specs === undefined) {
          specs = SpecialVisualizations.constructSpecification(txt)
          t[key] = specs
        }
      }
    } catch (e) {
      console.error("Could not construct a specification and with arguments", txt, "due to", e)
    }
  }

  function createVisualisation(specpart: Exclude<RenderingSpecification, string>): BaseUIElement {
    {
      try {
        return specpart.func
          .constr(state, tags, specpart.args, feature, layer)
          ?.SetClass(specpart.style)
      } catch (e) {
        console.error(
          "Could not construct a special visualisation with specification",
          specpart,
          "and tags",
          tags,
          "due to",
          e
        )
      }
    }
  }
</script>

{#if lang === "*"}
  {#each specs as specpart}
    {#if typeof specpart === "string"}
      <span class={clss}>
        {@html Utils.purify(Utils.SubstituteKeys(specpart, $tags))}
        <WeblateLink context={t.context} />
      </span>
    {:else if $tags !== undefined}
      <ToSvelte construct={() => createVisualisation(specpart)?.SetClass(clss)} />
    {/if}
  {/each}
{:else}
  <span {lang} class={clss}>
    {#each specs as specpart}
      {#if typeof specpart === "string"}
        <span>
          {@html Utils.purify(Utils.SubstituteKeys(specpart, $tags))}
          <WeblateLink context={t.context} />
        </span>
      {:else if $tags !== undefined}
        <ToSvelte construct={() => createVisualisation(specpart)} />
      {/if}
    {/each}
  </span>
{/if}
