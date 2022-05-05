<script>
	import Background from './components/Background.svelte';

	let scroll_container;

	function scrollTo(element) {
		const duration = 500;

		function helper(start_y, end_y, start_time, duration) {
			const current_time = new Date().getTime();

			let d = (current_time - start_time) / duration;
			d = 1 - Math.pow(1 - d, 3);

			if (d >= 1.0) {
				scroll_container.scrollTop = end_y;
			} else {
				let current_y = (end_y - start_y) * d + start_y;
				scroll_container.scrollTop = current_y;
				requestAnimationFrame(function() {
					helper(start_y, end_y, start_time, duration)
				});
			}
		}

		requestAnimationFrame(function () {
			const time = new Date().getTime();
			helper(scroll_container.scrollTop, element.scrollHeight, time, duration);
		});
	}

	function nextPage(e) {
		const target = document.getElementById(e.target.dataset.target);
		scrollTo(target);
	}

</script>

<style>
	h1 {
		font-weight: bold;
		font-size: 2.5rem;
        margin-bottom: 0rem;
	}

	p.tagline {
		font-size: 1.5rem;
        margin-top: 0rem;
	}

	div.scroll-container {
		width: 100%;
		height: 100%;
		overflow-x: hidden;
		overflow-y: scroll;
		scrollbar-width: none;
		-ms-overflow-style: none;
		pointer-events: none;
	}

	div.scroll-container::-webkit-scrollbar {
		display: none;
	}

	div.container {
		scroll-snap-align: start;
		display: flex;
		position: relative;
		width: 100%;
		height: 100vh;
		justify-content: center;
		align-items: center;
		flex-direction: column;
		z-index: 1;	
	}

    div.container > .title {
        text-shadow: 0px 0px 16px #ffffff20;
        margin-bottom: 2rem;
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
        filter: drop-shadow(0px 0px 16px #ffffff20);
	}

	div.social-icons > a.icon:hover {
		opacity: 0.5;
	}

	div.social-icons > a.discord {
		background-image: url('/assets/socials/discord.svg');
	}

	div.social-icons > a.github {
		background-image: url('/assets/socials/github.svg');
	}

	div.social-icons > a.twitch {
		background-image: url('/assets/socials/twitch.svg');
	}

	div.social-icons > a.twitter {
		background-image: url('/assets/socials/twitter.svg');
	}

	div.social-icons > a.youtube {
		background-image: url('/assets/socials/youtube.svg');
	}
</style>

<main>
    <Background scroll_container={scroll_container} />
    <div class="scroll-container" bind:this={scroll_container}>
        <div class="container">
            <div class="title" >
                <h1> Jack Bass </h1> 
                <p class="tagline" > Full Stack Software Engineer </p>
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
                <a class="icon discord" href="https://discordapp.com/users/126832247557980160">
                    <span class="sr-only">
                        my discord
                    </span>
                </a>
            </div>
        </div>
    </div>
</main>
