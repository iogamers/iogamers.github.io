<script>
	import { marked } from 'marked';

	let repos = yoinkRepos();

	async function yoinkRepos() { // i stole this from qmelz >:D
		const seasons = await fetch("https://api.github.com/orgs/iogamers/repos")
			.then(res => res.json())
			.then(repos => repos.filter(repo => repo.name.startsWith("season")))
	    
		const readmes = await Promise.all(seasons.map(repo => {
			return fetch(`https://api.github.com/repos/${repo.full_name}/readme`)
				.then(res => res.json())
				.then(json => fetch(json.download_url))
				.then(res => res.text())
		}));

		return seasons.map((it, i) => ({
			name: it.name,
			url: it.html_url,
			readme: readmes[i]
		}));
	}
</script>

{#await repos}
	yoinking seasons...
{:then repos}
	{#each repos as repo}
		<details class="season">
			<summary class="mc-font">{repo.name} / <a href={repo.url}>github</a></summary>
			<div class="readme">
				{@html marked.parse(repo.readme)}
			</div>
		</details>
	{/each}
{:catch err}
	{repos = yoinkRepos()}
{/await}