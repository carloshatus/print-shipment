<script>
  let files;
  let allItems;
  let pages = [];
  let maxPerPage = 6;

  function sliceIntoChunks(arr, chunkSize) {
    const res = [];
    for (let i = 0; i < arr.length; i += chunkSize) {
      const chunk = arr.slice(i, i + chunkSize);
      res.push(chunk);
    }
    return res;
  }

  function setItems(items) {
    allItems = items;
    pages = sliceIntoChunks(items, maxPerPage);
  }

  function setMaxItemsPerPage() {
    console.log(maxPerPage);
    pages = sliceIntoChunks(allItems, maxPerPage);
  }

  function print() {
    pages.forEach((page, index) => {
      window.print();
    });
  }

  $: if (files) {
    const read = new FileReader();
    read.readAsText(files[0]);
    read.onloadend = () => {
      setItems(JSON.parse(read.result));
    };
  }
</script>

<div class="noprint">
  <input type="file" bind:files />
  <input
    type="number"
    bind:value={maxPerPage}
    min="1"
    max="6"
    on:change={setMaxItemsPerPage}
  />
  <input
    type="range"
    bind:value={maxPerPage}
    min="1"
    max="6"
    on:change={setMaxItemsPerPage}
  />
  <button on:click={print}>Imprimir</button>
</div>
{#each pages as page, i}
  <div class="container" id={`page${i + 1}`}>
    {#each page as item, i}
      <div class="card">
        <p>{item.name}</p>
        <div>
          {#each item.details as detail, i}
            <div class="detail">
              <p>{detail.desc}: {detail.obs}</p>
              <p>
                {detail.value.toLocaleString("pt-BR", {
                  style: "currency",
                  currency: "BRL",
                })}
              </p>
            </div>
          {/each}
          {#if item.details.length > 1}
            <div class="detail">
              <p>Total:</p>
              <p>
                {item.details
                  .reduce((sum, curr) => sum + curr.value, 0)
                  .toLocaleString("pt-BR", {
                    style: "currency",
                    currency: "BRL",
                  })}
              </p>
            </div>
          {/if}
        </div>
        <p>{item.desc}</p>
      </div>
    {/each}
  </div>
{/each}

<style>
  .container {
    width: 21cm;
    height: 29.7cm;
    font-family: sans-serif;
    font-size: 1.2em;
    display: flex;
    flex-direction: row;
    flex-flow: wrap;
    justify-content: space-around;
    align-content: space-around;
    padding: 1cm;
  }
  .card {
    width: 7cm;
    height: 9cm;
    border: 1px solid;
    padding: 10px;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
  }
  .detail {
    padding-bottom: 15px;
  }
  @media print {
    .noprint {
      display: none;
    }
    .container {
      break-after: always;
      /* for firefox */
      page-break-after: always;
      /* for webkit */
      -webkit-column-break-after: always;
    }
  }
</style>
