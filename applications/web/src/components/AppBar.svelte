<script lang="ts">
  import fileDownload from "js-file-download"
  import {wasmProject, messageHistory} from "shared/stores"
  import type {Project} from "cadmium"
  import Download from "phosphor-svelte/lib/Download"
  import Upload from "phosphor-svelte/lib/Upload"
  import Bug from "phosphor-svelte/lib/Bug"
  import type {WithTarget} from "shared/types"
  import {isProject} from "shared/typeGuards"
  import {base} from "../base"
  import {renameProject} from "shared/projectUtils"

  const log = (function () { const context = "[AppBar.svelte]"; const color="gray"; return Function.prototype.bind.call(console.log, console, `%c${context}`, `font-weight:bold;color:${color};`)})() // prettier-ignore

  export let userName = "mattferraro.dev"
  export let project: Project
  export let renaming: boolean = false
  export let newProjectName: string = ""

  export let newFileContent: string | null = null

  $: project,
    (() => {
      // log("[project]", project)
      project && !isProject(project) && console.error("[AppBar.svelte] [project] fails isProject(project) typecheck", project)
    })()

  function fileInput(e: WithTarget<Event, HTMLInputElement>) {
    const target = e.target as HTMLInputElement
    const file = target.files![0]
    if (!file) return
    const reader = new FileReader()
    reader.onload = function (e) {
      // Note that this field is bound by the +page.svelte component,
      // which kicks off some changes as a result of this value changing.
      newFileContent = e.target?.result as string
    }
    reader.readAsText(file)
  }
</script>

<div class="bg-gray-200 h-[45px]">
  <!-- svelte-ignore a11y-click-events-have-key-events -->
  <div class="flex items-center gap-4 bg-gray-">
    <div class="shrink-0 select-none">
      <img class="object-cover h-10 w-10 ml-4" alt="logo" src="{base}/cadmium_logo_min.svg" />
    </div>
    <div class="select-none">CADmium</div>
    {#if renaming}
      <input
        class="bg-gray-300 text-gray-700 py-2 px-4 font-medium"
        type="text"
        bind:value={newProjectName}
        on:blur={() => {
          log("Renaming project aborted")
          renaming = false
          newProjectName = project.name ?? ""
        }}
        on:keydown={e => {
          if (e.key === "Enter") {
            log("Renaming project")
            renameProject(newProjectName)
            project.name = newProjectName
            renaming = false
          }
        }}
      />
    {:else}
      <!-- svelte-ignore a11y-no-static-element-interactions -->
      <div
        class="font-medium"
        on:dblclick={() => {
          log("Renaming project")
          renaming = true
          newProjectName = project.name ?? ""
        }}
      >
        {project.name ?? ""}
      </div>
    {/if}
    <!-- svelte-ignore a11y-no-static-element-interactions -->
    <div
      class="hover:bg-gray-300 rounded p-1"
      on:click={() => {
        let asString = $wasmProject.to_json()
        fileDownload(asString, `${project.name}.cadmium`)
      }}
    >
      <Download class="h-6 w-6" />
    </div>

    <!-- svelte-ignore a11y-no-static-element-interactions -->
    <div class="hover:bg-gray-300 rounded p-1">
      <!-- <Upload class="h-6 w-6" /> -->
      <!-- <input id="file-inp" type="file" style="visibility:hidden;" onchange="readFile(event)" /> -->
      <label for="file-inp">
        <Upload class="h-6 w-6" />
        <input id="file-inp" type="file" hidden on:change={fileInput} />
      </label>
    </div>

    <!-- svelte-ignore a11y-no-static-element-interactions -->
    <div
      class="hover:bg-gray-300 rounded p-1"
      on:click={() => {
        let asString = JSON.stringify($messageHistory)
        fileDownload(asString, `${project.name}.history.json`)
      }}
    >
      <Bug class="h-6 w-6" />
    </div>

    <div class="flex-grow flex flex-row-reverse gap-4 mr-4">
      <div>
        <a href="https://github.com/mattferraro/cadmium"><img class="h-6 w-6" src="{base}/github-mark.svg" alt="github logo" /></a>
      </div>
      <div>{userName}</div>
    </div>
  </div>
</div>
