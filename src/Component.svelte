<script>
  import { getContext, onDestroy, onMount } from "svelte";
  import Tags from "./Tags.svelte"

  export let tagsField;

  export let showLabel
  export let labelText;
  export let placeholder;

  export let maxTags;
  export let onlyUnique;
  export let allowBlur;
  export let disabled;
  export let readonly;

  export let acData
  export let acLabelField
  export let forceAutocomplete

  export let renderTagArray

  let tags
  let fieldApi;
  let fieldState;

  const {styleable} = getContext("sdk");
  const component = getContext("component");
  const formContext = getContext("form");
  const formStepContext = getContext("form-step");
  const fieldGroupContext = getContext("field-group");

  const formApi = formContext?.formApi;
  const labelPos = fieldGroupContext?.labelPosition || "above";

  $: formStep = formStepContext ? $formStepContext || 1 : 1;
  $: formField = formApi?.registerField(tagsField, "text", "", false, null, formStep);

  $: unsubscribe = formField?.subscribe((value) => {
    fieldState = value?.fieldState;
    fieldApi = value?.fieldApi;
  });

  onMount(() => {
    tags = initTags(fieldState?.value);
  })

  onDestroy(() => {
    fieldApi?.deregister();
    unsubscribe?.();
  });

  $: autocomplete = initAutocomplete(acData.rows);
  $: labelClass = (labelPos === "above") ? "" : `spectrum-FieldLabel--${labelPos}`;
  $: maxTagCount = (maxTags <= 0) ? false : maxTags;
  $: onlyUnique = onlyUnique || false;

  function initTags(d) {
    let tags = [];
    if (d !== "" || d.length > 0) {
      let tag = d.split(",");
      tag.forEach(t => {
        tags.push(t.trim());
      })
    }
    return tags;
  }

  function initAutocomplete(d) {
    let autocompleteObjects = [];
    d.forEach((row) => {
      if (row[acLabelField] !== undefined) {
        autocompleteObjects.push(row[acLabelField]);
      }
    });
    return autocompleteObjects;
  }

  function tagChange() {
    fieldApi?.setValue(tags.join(','))
  }

  function tagAdd() {}

  function tagRemove() {}

  function tagClick(tag) {}
</script>

<div class="spectrum-Form-item" use:styleable={$component.styles}>
  {#if !formContext}
    <div class="placeholder">Form components need to be wrapped in a form</div>
  {:else}
    {#if !showLabel}
      <label
              class:hidden={!labelText}
              htmlFor={fieldState?.fieldId}
              class={`spectrum-FieldLabel spectrum-FieldLabel--sizeM spectrum-Form-itemLabel ${labelClass}`}
      >
        {labelText || " "}
      </label>
    {/if}
    <div class="spectrum-Form-itemField">
      {#if !tagsField}
        <div class="error">Please select a field</div>
      {:else}
        <Tags
                bind:tags={tags}
                maxTags={maxTagCount}
                onlyUnique={onlyUnique}

                allowBlur={allowBlur}
                disable={disabled}
                readonly={readonly}

                labelText={labelText}
                labelShow={showLabel}
                placeholder={placeholder}

                autoComplete={autocomplete}
                onlyAutocomplete={forceAutocomplete}

                onTagChange={tagChange}
                onTagAdd={tagAdd}
                onTagRemove={tagRemove}
                onTagClick={tagClick}
        />
        {#if fieldState?.error}
          <div class="error">{fieldState.error}</div>
        {/if}
      {/if}
    </div>
  {/if}
</div>
