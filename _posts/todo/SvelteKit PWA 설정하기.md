1. Yarn 추가
	```zsh
	yarn add -D vite-plugin-pwa @vite-pwa/sveltekit
	```
1. vite.config.ts에 설정 적용하기
```ts
import { sveltekit } from '@sveltejs/kit/vite';
import { defineConfig } from 'vite';
import { SvelteKitPWA } from '@vite-pwa/sveltekit';

export default defineConfig({
	plugins: [
		sveltekit(),
		SvelteKitPWA({
			srcDir: './src',
			// mode: 'development',
			mode: 'production',
			// you don't need to do this if you're using generateSW strategy in your app
			// strategies: generateSW ? 'generateSW' : 'injectManifest',
			strategies: 'generateSW',
			// you don't need to do this if you're using generateSW strategy in your app
			// filename: generateSW ? undefined : 'prompt-sw.ts',
			filename: undefined,
			scope: '/',
			base: '/',
			selfDestroying: process.env.SELF_DESTROYING_SW === 'true',
			manifest: {
				short_name: 'Toast',
				name: 'Toast - 오늘의 출근',
				start_url: '/',
				scope: '/',
				display: 'standalone',
				theme_color: '#070709',
				background_color: '#070709',
				icons: [
					{
						src: '/android-chrome-192x192.png',
						sizes: '192x192',
						type: 'image/png'
					},
					{
						src: '/android-chrome-512x512.png',
						sizes: '512x512',
						type: 'image/png'
					},
					{
						src: '/android-chrome-512x512.png',
						sizes: '512x512',
						type: 'image/png',
						purpose: 'any maskable'
					}
				]
			},
			injectManifest: {
				globPatterns: ['client/**/*.{js,css,ico,png,svg,webp,woff,woff2}']
			},
			workbox: {
				globPatterns: ['client/**/*.{js,css,ico,png,svg,webp,woff,woff2}']
			},
			devOptions: {
				enabled: true,
				suppressWarnings: process.env.SUPPRESS_WARNING === 'true',
				type: 'module',
				navigateFallback: '/'
			},
			// if you have shared info in svelte config file put in a separate module and use it also here
			kit: {}
		})
	]
});

```
3. +layout에 manifest 등록 & tsconfig 수정
```html
<!-- src/routes/+layout.svelte -->
<script>
  import { pwaInfo } from 'virtual:pwa-info'; 

  $: webManifestLink = pwaInfo ? pwaInfo.webManifest.linkTag : '' 
</script> 
  
<svelte:head> 
 	{@html webManifestLink} 
</svelte:head>
```
```json
// tsconfig.json
{
	"compilerOptions": {
		"types": [
			"vite-plugin-pwa/info"
		]
	}
}
```

4. 업데이트 설정하기
-  Auto
```html
<script>
  import { onMount } from 'svelte'
  import { pwaInfo } from 'virtual:pwa-info'
  
  onMount(async () => {
    if (pwaInfo) {
      const { registerSW } = await import('virtual:pwa-register')
      registerSW({
        immediate: true,
        onRegistered(r) {
          // uncomment following code if you want check for updates
          // r && setInterval(() => {
          //    console.log('Checking for sw update')
          //    r.update()
          // }, 20000 /* 20s for testing purposes */)
          console.log(`SW Registered: ${r}`)
        },
        onRegisterError(error) {
          console.log('SW registration error', error)
        }
      })
    }
  })
  
  $: webManifest = pwaInfo ? pwaInfo.webManifest.linkTag : ''
</script>

<svelte:head>
    {@html webManifest}
</svelte:head>

<main>
  <slot />
</main>
```
- Prompt
```html
<!-- src/lib/ReloadPrompt.svelte -->
<script lang="ts">
	import { useRegisterSW } from 'virtual:pwa-register/svelte'
	const {
		needRefresh,
		updateServiceWorker,
		offlineReady
	} = useRegisterSW({
		onRegistered(r) {
		// uncomment following code if you want check for updates
		// r && setInterval(() => {
		//    console.log('Checking for sw update')
		//    r.update()
		// }, 20000 /* 20s for testing purposes */)
			console.log(`SW Registered: ${r}`)
		},
		onRegisterError(error) {
			console.log('SW registration error', error)
		},
	})
	const close = () => {
		offlineReady.set(false)
		needRefresh.set(false)
	}
	$: toast = $offlineReady || $needRefresh
</script>

{#if toast}
	<div class="pwa-toast" role="alert">
		<div class="message">
			{#if $offlineReady}
				<span>
					App ready to work offline
				</span>
			{:else}
				<span>
					New content available, click on reload button to update.
				</span>
			{/if}
		</div>
		{#if $needRefresh}
			<button on:click={() => updateServiceWorker(true)}>
				Reload
			</button>
		{/if}
		<button on:click={close}>
			Close
		</button>
	</div>
{/if}

<style>
	.pwa-toast {
		position: fixed;
		right: 0;
		bottom: 0;
		margin: 16px;
		padding: 12px;
		border: 1px solid #8885;
		border-radius: 4px;
		z-index: 2;
		text-align: left;
		box-shadow: 3px 4px 5px 0 #8885;
		background-color: white;
	}
	.pwa-toast .message {
		margin-bottom: 8px;
	}
	.pwa-toast button {
		border: 1px solid #8885;
		outline: none;
		margin-right: 5px;
		border-radius: 2px;
		padding: 3px 10px;
	}
</style>
```

```html
<script>
  import { pwaInfo } from 'virtual:pwa-info'

  $: webManifest = pwaInfo ? pwaInfo.webManifest.linkTag : ''  
</script>

<svelte:head>
    {@html webManifest}
</svelte:head>

<main>
  <slot />
</main>

{#await import('$lib/ReloadPrompt.svelte') then { default: ReloadPrompt}}
  <ReloadPrompt />
{/await}
```

5. Build & Test
```zsh
yarn build
yarn preview
```

참고
https://vite-pwa-org.netlify.app/frameworks/sveltekit.html
https://github.com/vite-pwa/sveltekit/tree/main/examples/sveltekit-ts