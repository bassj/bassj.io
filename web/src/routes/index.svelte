<script>
	import Background from '../components/Background.svelte';
</script>

<style>
	h1 {
		font-weight: bold;
		font-size: 2.5rem;
	}

	h2 {
		font-size: 1.5rem;
	}

	div.container {
		display: flex;
		width: 100%;
		height: 100%;
		justify-content: center;
		align-items: center;
		flex-direction: column;
		z-index: 1;
		pointer-events: none;
	}

	div.social-icons {
		display: flex;
		justify-content: center;
		align-items: center;

	}
	
	div.social-icons > a.icon {
		pointer-events: all;
		width: 3rem;
		height: 3rem;
		margin-left: 0.5rem;
		margin-right: 0.5rem;
		content: none;
		background-size:contain;
		background-position: center center;
		background-repeat: no-repeat;
		transition: opacity 0.2s;
	}

	div.social-icons > a.icon:hover {
		opacity: 0.5;
	}

	div.social-icons > a.discord {
		background-image: url('/socials/discord.svg');
	}

	div.social-icons > a.github {
		background-image: url('/socials/github.svg');
	}

	div.social-icons > a.twitch {
		background-image: url('/socials/twitch.svg');
	}

	div.social-icons > a.twitter {
		background-image: url('/socials/twitter.svg');
	}

	div.social-icons > a.youtube {
		background-image: url('/socials/youtube.svg');
	}

</style>

<svelte:head>
</svelte:head>

<Background />

<div class="container">
	<div>
		<h1> Jack Bass </h1> 
		<h2> Full Stack Software Engineer </h2>
	</div>
	<div class="social-icons">
		<a class="icon github" href="https://github.com/bassj">
			<span class="sr-only">
				my github
			</span>
		</a>
		<a class="icon youtube" href="https://www.youtube.com/channel/UCR_llcHj7BNvzwkj95AhRNQ">
			<span class="sr-only">
				my youtube
			</span>
		</a>
		<a class="icon twitch" href="https://www.twitch.tv/bassjio">
			<span class="sr-only">
				my twitch
			</span>
		</a>
		<a class="icon twitter" href="https://twitter.com/bassjio">
			<span class="sr-only">
				my twitter
			</span>
		</a>
		<a class="icon discord" href="/">
			<span class="sr-only">
				my discord
			</span>
		</a>

	</div>
</div>