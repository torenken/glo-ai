<script lang="ts">
	import Message from '$lib/components/Message.svelte';
	import type { ChatCompletionRequestMessage } from 'openai';
	import { SSE } from 'sse.js';

	let query = '';
	let answer = '';

	let showChat = false;
	let loading = false;
	let chatMessages: ChatCompletionRequestMessage[] = [];

	const handleSubmit = async (selectedText?: string) => {
		if (selectedText) {
			query = 'What is ' + selectedText;
		}

		loading = true;
		chatMessages = [...chatMessages, { role: 'user', content: query }];

		const eventSource = new SSE('/api/message', {
			headers: {
				'Content-Type': 'application/json'
			},
			payload: JSON.stringify({ messages: chatMessages })
		});
		query = '';

		eventSource.addEventListener('error', handleError);
		eventSource.addEventListener('message', (e) => {
			try {
				loading = false;
				showChat = true;

				if (e.data === '[DONE]') {
					chatMessages = [...chatMessages, { role: 'assistant', content: answer }];
					answer = '';
					return;
				}
				const completionResponse = JSON.parse(e.data);
				const [{ delta }] = completionResponse.choices;

				if (delta.content) {
					answer = (answer ?? '') + delta.content;
				}
			} catch (err) {
				handleError(err);
			}
		});
		eventSource.stream();
	};

	function handleError<T>(err: T) {
		loading = false;
		query = '';
		answer = '';
		console.error(err);
	}
</script>

<div class="flex flex-col pt-4 w-full px-8 items-center gap-2">
	<div>
		<h1 class="text-4xl font-bold w-full text-center">Welcome to GloAI</h1>
		<p class="text-sm italic">Powered by gpt-3.5-turbo</p>
	</div>
	<div class="text-lg italic">
		<p>
			Nachdem <a href='/' class="link" on:click|preventDefault={() => handleSubmit('Chat-GPT')}>ChatGPT</a> am
			30. November 2022 für die Öffentlichkeit frei zugänglich geworden war, meldeten sich innerhalb
			von fünf Tagen eine Million Nutzer an; hingegen hatten Instagram erst nach zweieinhalb Monaten
			und Spotify erst nach fünf Monaten eine Million Nutzer. Im Januar 2023 erreichte ChatGPT bereits
			über 100 Millionen Nutzer, womit es bis dato die mit Abstand am schnellsten wachsende Verbraucher-Anwendung
			ist.
		</p>
		<p>
			ChatGPT soll schon von Anbeginn im Vergleich zum Vorgänger
			<a href='/' class="link" on:click|preventDefault={() => handleSubmit('InstructGPT')}>InstructGPT</a>
			darauf angelegt worden sein, schädliche und irreführende Antworten zu vermeiden. Während InstructGPT
			noch die Vorgabe in der Anfrage „Erzähle mir davon, wie Christoph Kolumbus 2015 in die USA kam“
			als wahr wertet, nutzt ChatGPT Wissen über Kolumbusʼ Reisen und das bis zum Jahre 2021 erlernte
			Verständnis, um eine Antwort zu liefern, die annimmt, was geschehen wäre, wenn Kolumbus 2015 in
			den USA gelandet wäre.
		</p>
		<p>
			Mit Updates vom 15. Dezember 2022 und 9. Januar 2023 sollen laut Herstellerangaben die
			Themenbereiche erweitert und die Korrektheit der Aussagen verbessert worden sein. Ein Update
			am 30. Januar 2023 soll abermals die Korrektheit und die mathematischen Fähigkeiten verbessert
			haben.
		</p>
		<p>
			Im Februar 2023 hielt der NEOS-Abgeordnete Niko Swatek im Landtag Steiermark eine Rede zum
			Thema Schulstraßen, die er von ChatGPT schreiben ließ, wie er nach zwei weiteren Rednern
			bekanntgab. Die erste von ChatGPT geschriebene Rede im Europaparlament hielt der
			Volt-Abgeordnete Damian Boeselager im Februar 2023. Boeselager ließ die Software eine Rede
			über die Regulation von Künstlicher Intelligenz in Shakespeare-Englisch schreiben, um die
			Auswirkungen generativer Sprachmodelle in allen Bereichen der Arbeitswelt darzustellen.
		</p>
		<p class="text-sm italic pt-3">Quelle: https://de.wikipedia.org/wiki/ChatGPT</p>
	</div>

	{#if showChat}
		<div class="h-[700px] w-full bg-gray-800 p-4 overflow-y-auto flex flex-col gap-4">
			<div class="flex flex-col gap-2">
				<Message type="assistant" message="Hello, click on a link or ask me anything you want!" />
				{#each chatMessages as message}
					<Message type={message.role} message={message.content} />
				{/each}
				{#if answer}
					<Message type="assistant" message={answer} />
				{/if}
				{#if loading}
					<Message type="assistant" message="Loading.." />
				{/if}
			</div>
		</div>

		<form
			class="flex w-full rounded-md gap-8 bg-gray-800 p-2"
			on:submit|preventDefault={() => handleSubmit()}
		>
			<input type="text" class="input input-bordered w-full" bind:value={query} />
			<button type="submit" class="btn btn-accent">Ask Me</button>
		</form>

		<div class='w-full text-left text-lg italic '>
			<p class="text-sm italic pt-3">Credits to Hunter Johnston (https://github.com/huntabyte/chatty)</p>
		</div>
	{/if}
</div>
