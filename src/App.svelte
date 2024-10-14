<script>
  import { onMount } from "svelte";
  let dictionary = {};
  let scrambledWord = "";
  let results = [];

  async function loadDictionary() {
    let d = {};
    try {
      const response = await fetch("US_4_final.txt");
      const text = await response.text();
      const dicList = text.split("\n");
      dicList
        .filter((word) => word !== "")
        .forEach((word) => {
          let intVal = wordToInt(word);
          if (!d[intVal]) {
            d[intVal] = [];
          }
          d[intVal].push(word);
        });
    } catch (error) {
      console.log("No Dictionary!", error);
    }
    return d;
  }

  function wordToInt(word) {
    const LETTERS = "abcdefghijklmnopqrstuvwxyz".split("");
    let wordInt = BigInt(0);
    for (let letter of word.toLowerCase()) {
      let i = LETTERS.indexOf(letter);
      if (i === -1) continue;
      wordInt += BigInt(2) ** (BigInt(4) * BigInt(i));
    }
    return wordInt;
  }

  function findMatches() {
    let wordResults = [];
    for (let i = 3; i <= scrambledWord.length; i++) {
      let combinations = getCombinations(scrambledWord, i);

      combinations.forEach((subset) => {
        let subsetStr = subset.join("");
        let v = wordToInt(subsetStr);

        if (dictionary[v]) {
          wordResults.push(...dictionary[v]);
        }
      });
    }
    results = Array.from(new Set(wordResults));
  }

  function getCombinations(word, len) {
    if (len === 1) return word.split("").map((char) => [char]);
    let combs = [];
    for (let i = 0; i < word.length; i++) {
      let restCombs = getCombinations(word.slice(i + 1), len - 1);
      restCombs.forEach((comb) => {
        combs.push([word[i], ...comb]);
      });
    }
    return combs;
  }

  onMount(async () => {
    dictionary = await loadDictionary();
  });
</script>

<div>
  <form on:submit|preventDefault={findMatches}>
    <input
      type="text"
      bind:value={scrambledWord}
      placeholder="Enter scrambled word (ENGLISH ONLY)"
    />
    <hr />
    <button>Find Words</button>
  </form>

  <div>
    {#if results.length > 0}
      <h3>Possible Words ({results.length}):</h3>
      <ul>
        {#each results as result}
          <li>{result}</li>
        {/each}
      </ul>
    {/if}
  </div>
</div>

<style>
  ul {
    text-align: left;
    columns: 3;
    -webkit-columns: 2;
    -moz-columns: 2;
  }

  input {
    padding: 8px;
    border-radius: 8px;
    border-style: solid;
    width: 90%;
  }
</style>
