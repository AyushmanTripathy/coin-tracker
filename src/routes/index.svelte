<script>
  let coins = [];
  let watching = ["bitcoin", "Tether", "Ethereum"];
  let names = [];

  async function getNames() {
    let res = await fetch("").catch(console.error);
    return await res.json();
  }

  getPrices();
  async function getPrices() {
    let res = await fetch(
      `https://api.coingecko.com/api/v3/coins/markets?vs_currency=inr&order=market_cap_desc&per_page=30&page=1&sparkline=false&price_change_percentage=24h`
    ).catch(console.error);
    res = await res.json();

    const formatter = new Intl.NumberFormat("en-IN", {});
    for (const coin of res) {
      coin.price_change_percentage_24h =
        Math.round(coin.price_change_percentage_24h * 100) / 100;
      coin.current_price = formatter.format(coin.current_price);
    }
    coins = res;
  }
  function addName(name) {
    watching.push(name);
    getPrices();
  }

  let lastSearched;
  function search({ srcElement }) {
    const index = coins.find((coin) =>
      coin.name.toLowerCase().includes(srcElement.value)
    );
    if (lastSearched) lastSearched.style.color = "#bbb";
    if (!index) return;
    lastSearched = document.getElementById(index.name);
    lastSearched.style.color = "yellow";
    lastSearched.scrollIntoView();
  }
  function handleClick({ srcElement }) {
    location.href = "/" + srcElement.parentNode.classList[0];
  }
</script>

<main>
  <input type="text" placeholder="search a coin" on:input={search} />
  <section>
    <div>
      <td>Rank</td>
      <td>Coin</td>
      <td>value</td>
      <td>market cap</td>
      <td>change in 24h</td>
    </div>
    {#each coins as { id, name, image, market_cap_rank, current_price, price_change_percentage_24h, market_cap }}
      <div id={name} class={id} on:click={handleClick}>
        <td>{market_cap_rank}</td>
        <td> <img src={image} alt={name} /> {name} </td>
        <td> ₹ {current_price}</td>
        <td> ₹ {market_cap}</td>
        <td>
          <span
            class="material-icons-outlined {price_change_percentage_24h < 0
              ? 'down'
              : 'up'}"
          >
            arrow_{price_change_percentage_24h < 0 ? "downward" : "upward"}
          </span>
          {price_change_percentage_24h}</td
        >
      </div>
    {/each}
  </section>
</main>

<style lang="scss">
  main {
    @include absolute;
    @include flex(column);
    @include flex-center;
    @include fullscreen;

    color: $light;
    background-color: $pri;
  }
  input {
    @include section(5vh, 30vw);
    outline: none;
    background-color: $sec;
    border: none;
    color: $light;
    font-size: 16px;
    padding: 8px;
    margin-bottom: 2vh;
  }
  img {
    @include section(16px, 16px);
    padding-right: 5px;
  }
  section {
    @include section(80vh, 100vw);
    max-width: 1080px;

    border-collapse: collapse;
    box-shadow: 0px 8px 16px 0px rgba(0, 0, 0, 0.3);
    white-space: nowrap;
    overflow: auto;
    overflow-x: scroll;
  }
  td,
  th {
    @include flex(row);
    align-items: center;
    border-bottom: 1px solid $light;
    text-align: left;
    padding: 8px;
  }
  div {
    @include grid(1fr 1fr 1fr 1fr 1fr, 1fr);
    width: 100%;
    cursor: pointer;
  }
  div:nth-child(even) {
    background-color: $sec;
  }
</style>
