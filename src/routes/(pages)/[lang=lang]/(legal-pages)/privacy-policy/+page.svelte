<script lang="ts">
	import LL from '$i18n/i18n-svelte';
	import LegalLayout from 'components/LegalLayout.svelte';
</script>

<LegalLayout title={$LL.pages.privacy.title()} content={$LL.pages.privacy.content()}></LegalLayout>
