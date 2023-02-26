<!-- YOU CAN DELETE EVERYTHING IN THIS PAGE -->

<script lang="ts">
	import { ProgressRadial } from '@skeletonlabs/skeleton';
import { onMount } from 'svelte';

	type TGameState = "loading" | "playing" | "finished" | "gameover" 
	let gameState: TGameState = "loading"; 

	let gameoverReason: string = ""
	let wordInvalid: boolean = false
	let invalidReason: "wrong" | "guessed" | "short" | "" = ""

	let wordOfTheDay: string = "";
	let maxGuesses: number = 6 
	let guessIndex: number = 6;
	let guesses: string[][] = []
	let capture: string = ""
	let timer: number = -1;
	let timerRunning: boolean = false;
	let timerIncrease = 15

	let keyboard: string[] = [
		"qwertyuiop",
		'asdfghjkl',
		'zxcvbnm'
	]

	const setup = () => {
		for (let i = 0; i < maxGuesses; i++) {
			let row = []
			for(let j = 0; j < 5; j++) {
				row.push('')
			}
			guesses.push(row)
		}
		guesses = guesses
		
		fetch("https://random-word-api.vercel.app/api?words=1&length=5")
			.then(response => response.json())
			.then(result => {
				gameState = "playing"
				wordOfTheDay = result[0]
				timer = 30
				document.getElementById('give-up')?.addEventListener('click', () => {
					console.log("Giving up")
					giveUp()
				})
			})
			.catch(error => console.log('error', error));


	}

	const clampWord = () => {
		wordInvalid = false
		capture = capture.slice(0, 5)
	}

	const guess = async () => {
		let check = await fetch(`https://api.dictionaryapi.dev/api/v2/entries/en/${capture}`)
		wordInvalid = false
		
		if (check.status === 404) {
			wordInvalid = true
			invalidReason = "wrong"
		}

		guesses.forEach(word => {
			if (word.join('') === capture) {
				wordInvalid = true
				invalidReason = "guessed"
			}
		})

		if (capture.length !== 5) {
			wordInvalid = true
			invalidReason = "short"
		}

		if (wordInvalid) return

		wordInvalid = false
		timerRunning = true
		
		guesses.unshift(guesses.pop() ?? [])

		for (let i = 0; i < capture.length; i++) {
			guesses[0][i] = capture[i]
		}
		guesses = guesses
		guessIndex--

		if (capture.toLowerCase() === wordOfTheDay?.toLowerCase()) {
			gameState = "finished"
			return
		}

		capture = ""
		timer += timerIncrease

		if (guessIndex === 0) {
			gameState = "gameover"
			gameoverReason = "You ran out of guesses!"
			return
		}
	}

	const checkLetter = (row: number, col: number) => {
		let delay = `${col * 100}`
		if (wordOfTheDay && guesses[row][col].toLowerCase() === wordOfTheDay[col].toLowerCase()) 
			return `variant-filled-primary scale-110 delay-${delay}`

		if (wordOfTheDay?.includes(guesses[row][col].toLowerCase())) 
			return `variant-ghost-warning scale-110 delay-${delay}`
		
		return `variant-soft-surface`
	}

	$: letterGuessed = (letter: string) => {
		for (let i = 0; i < guesses.length; i++) {
			for (let j = 0; j < guesses[i].length; j++) {
				if (letter.toLowerCase() === guesses[i][j].toLowerCase()) {
					if (wordOfTheDay.toLowerCase().includes(letter.toLowerCase())) {
						return 'variant-ghost-primary'
					} else {
						return 'variant-ringed-surface  opacity-10'
					}
				}
			}
		}
		return 'variant-ghost-surface'
	}

	const guessWithKeyboard = (letter: string) => {
		if (capture.length < 5) {
			capture += letter
		}
	}

	const giveUp = () => {
		gameState = "gameover"
		gameoverReason = "You gave up!"
	}
	
	setInterval(() => {
		if (!timerRunning || gameState !== 'playing') 
            return

		if (timer > 0)
            timer--

		if (timer === 0) {
			gameState = "gameover"
            gameoverReason = "You ran out of time!"
        }
	}, 1000);

	const guessOnEnter = (event: KeyboardEvent) => {
		if (event.key === 'Enter') {
			event.preventDefault()
			guess()
		}
	}

	const getTimerColor = (timer: number) => {
		if (timer < 10) return 'text-error-500'
		if (timer < 30) return 'text-warning-500'
		return 'text-primary-500'
	}

	setup()
</script>
<div class="container h-full mx-auto flex flex-col justify-center items-center py-10">
	{#if gameState === "loading"}
		<h1>Loading...</h1>
		<ProgressRadial class="w-40 h-40 my-8 stroke-primary-500" value={undefined} />
	{:else if gameState === "playing"}

	<div class="mt-8 text-center">
		<h2>
			<span class={`${getTimerColor(timer)}`}>{timer}</span> SECONDS LEFT
		</h2>
		{#if !timerRunning}
			<span>
				The timer will start after your first guess and each guess will increase the timer by {timerIncrease} seconds.
			</span>
		{/if}
	</div>
	<div class="flex flex-col justify-center items-center mt-8">
		{#each keyboard as row}
			<div class="flex">
				{#each row as letter}
					<button 
						class={`card w-6 h-6 md:w-8 md:h-8 flex justify-center items-center m-0.5 duration-200 ${letterGuessed(letter)}`}
						on:click={() => guessWithKeyboard(letter)}
					>
						{letter.toUpperCase()}
					</button>
				{/each}
			</div>
		{/each}
	</div>
	{#if wordInvalid}
		<h4 class="text-red-500 mt-4">
			{#if invalidReason === 'wrong'}"{capture}" isn't a word!{/if}
			{#if invalidReason === 'guessed'}"{capture}" has already been guessed!{/if}
			{#if invalidReason === 'short'}"{capture}" is too short!{/if}
		</h4>
	{/if}	
	<form on:submit={guess}>
		<div class="input-group input-group-divider grid-cols-[1fr_auto] mt-4 max-w-[300px]">
			<input type="text" class="input" bind:value={capture} on:input={clampWord} on:keypress={guessOnEnter} />
			<div class="variant-filled-tertiary duration-200 whitespace-nowrap">{guessIndex} Left</div>
		</div>
		<button type="submit" class={`btn variant-filled-secondary duration-200 whitespace-nowrap mt-4 w-[100%] max-w-[300px]`} disabled={capture.length !== 5} on:click={guess}>Guess {capture}</button>
	</form>

		<div class="grid grid-cols-5 gap-4 mt-8">
			{#each guesses as word, row}
				{#each word as letter, col}
					<div class={`card w-12 h-12 md:w-16 md:h-16 lg:w-20 lg:h-20 flex justify-center items-center transform-gpu duration-200 ${letter==="" ? 'variant-ringed-surface' : checkLetter(row, col)}`}>
						<h2>{letter.toUpperCase()}</h2>
					</div>
				{/each}
			{/each}

			
		</div>

		
	{:else if gameState === "finished"}
		<h3 class="text-primary-500">You won in {6 - guessIndex} guesses!</h3>
	{:else}
		<h3 class="text-red-500">{gameoverReason}</h3>
	{/if}
	{#if gameState === "gameover" ||  gameState === "finished"}
		<h1>The word was <span class="text-primary-500">{wordOfTheDay}</span>!</h1>
		<button class="btn variant-filled-primary mt-4" on:click={() => window.location.reload()}>Try Again</button>
	{/if}
	
</div>
