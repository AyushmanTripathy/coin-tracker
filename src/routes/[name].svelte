<script>
  import { onMount } from "svelte";
  import { page } from "$app/stores";

  let data = {
    inr_market_cap : 0,
    inr_24hr_change : 0,
    inr_24h_vol : 0,
    inr: 0,
  };
  const name = $page.params.name;

  getData();
  async function getData() {
    let res = await fetch(
      `https://api.coingecko.com/api/v3/simple/price?ids=${name}&vs_currencies=inr&include_market_cap=true&include_24hr_vol=true&include_24hr_change=true`
    ).catch(console.error);
    res = await res.json();
    res = res[name];

    const formatter = new Intl.NumberFormat("en-IN", {});
    for (const key in res) res[key] = formatter.format(res[key]);
    data = res;
  }

  onMount(async (promise) => {
    let res = await fetch(
      `https://api.coingecko.com/api/v3/coins/${name}/market_chart?vs_currency=inr&days=365&interval=daily`
    );
    res = await res.json();

    const labels = res.prices.map((ele) => {
      return new Date(ele[0])
        .toISOString()
        .replace("-", "/")
        .split("T")[0]
        .replace("-", "/");
    });
    for (const name in res) {
      res[name] = res[name].map((ele) => Math.round(ele[1]));
    }

    const year_data = {
      labels,
      datasets: [
        {
          label: "Value",
          pointStrokeColor: "#bbb",
          pointRadius: 1,
          pointHighlightFill: "#bbb",
          data: res.prices,
          backgroundColor: ["#bbb"],
          borderColor: ["#bbb"],
          backgroundColor: "#bbb",
          borderWidth: 1,
          fill: false,
        },
      ],
      options: {
        scales: {
          xAxes: [
            {
              type: "time",
              distribution: "linear",
            },
          ],
        },
      },
    };
    new Chart(document.getElementById("year_chart").getContext("2d"), {
      type: "line",
      data: year_data,
    });
  });
</script>

<main>
  <a href="/">
    <span id="back" class="material-icons-outlined"> arrow_back_ios </span>
  </a>

  <h1>{name}</h1>
  <h2>₹{data.inr}</h2>
  <p>market cap : ₹{data.inr_market_cap}</p>
  <p>
    volume : ₹{data.inr_24h_vol}
    <span class="{data.inr_24h_change > 0 ? "green" : "red"}">
      ({Math.round(data.inr_24h_change * 100) / 100})
    </span>
  </p>

  <section>
    <canvas id="year_chart" />
  </section>
</main>

<style lang="scss">
  main {
    @include fullscreen;
    @include absolute;
    @include flex(column);
    @include flex-center;

    background-color: $pri;
    color: $light;
  }
  section {
    @include section(fit-content, 80vw);
  }
  h1 {
    border-bottom:1px solid $light;
  }
  .red {
    color: red;
  }
  .green {
    color: green;
  }
  #back {
    @include fixed;
    top: 5vh;
    left: 5vw;

    color: $light;
    cursor: pointer;
    padding: 20px;

    border-radius: 50%;
    box-shadow: 0px 8px 16px 0px rgba(0, 0, 0, 0.3);
  }
  canvas {
    box-shadow: 0px 8px 16px 0px rgba(0, 0, 0, 0.3);
  }
</style>
