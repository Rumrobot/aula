<script lang="ts">
  import { afterNavigate, goto } from "$app/navigation";
  import { authStore } from "$lib/stores";
  import { get, writable } from "svelte/store";

  export let redirectIfFail: string = "/log-ind";
  export let whitelisted: string[] = ["/log-ind"];

  const showContent = writable(false);
  afterNavigate(async () => {
    if (whitelisted.includes(window.location.pathname)) {
      return showContent.set(true);
    }
    const { cookie } = get(authStore);
    if (cookie) {
      const response = await fetch("https://api.betterlectio.dk/check-cookie", {
        headers: {
          "lectio-cookie": cookie,
        },
      });
      const { valid } = await response.json();
      console.log(valid);
      if (!valid) {
        if ($authStore.username != "" && $authStore.password != "") {
          const response = await fetch("https://api.betterlectio.dk/auth", {
            headers: {
              brugernavn: $authStore.username,
              adgangskode: $authStore.password,
              skole_id: String($authStore.school),
            },
          });
          if (response.ok) {
            $authStore.cookie = response.headers.get("set-lectio-cookie") ?? "";
          } else {
            $authStore.username = "";
            $authStore.password = "";
            $authStore.cookie = "";
            return goto(redirectIfFail);
          }
        } else {
          $authStore.cookie = "";
          return goto(redirectIfFail);
        }
      }
    } else {
      return goto(redirectIfFail);
    }
    showContent.set(true);
  });
</script>

{#if $showContent}
  <slot />
{:else}
  <div class="flex flex-col items-center justify-center">
    <h1 class="text-2xl font-bold">Loading</h1>
    <p class="text-gray-500">Please wait while we check your credentials</p>
  </div>
{/if}
