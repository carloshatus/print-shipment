<script lang="ts">
  type ItemDetail = {
    desc?: string;
    obs?: string;
    value: number;
  };

  type Item = {
    name: string;
    details: ItemDetail[];
    desc?: string;
  };

  type Page = {
    print?: boolean;
    items: Item[];
  };

  let files: FileList;
  let allItems: Item[];
  let pages: Page[] = [];
  let maxPerPage = 6;

  function sliceIntoChunks(arr: Item[], chunkSize: number): Page[] {
    const res = [];
    for (let i = 0; i < arr.length; i += chunkSize) {
      const chunk = arr.slice(i, i + chunkSize);
      res.push({ items: chunk });
    }
    return res;
  }

  function setItems(items: Item[]): void {
    allItems = items;
    pages = sliceIntoChunks(items, maxPerPage);
  }

  function setMaxItemsPerPage(): void {
    console.log(maxPerPage);
    pages = sliceIntoChunks(allItems, maxPerPage);
  }

  function printPage(index: number): void {
    pages = [
      ...pages.map((page, i) => {
        if (i !== index) {
          page.print = false;
        } else {
          page.print = true;
        }
        return page;
      }),
    ];
    setTimeout(() => {
      window.print();
    }, 500);
  }

  function downloadExample() {
    const exampleJson = [
      {
        name: "Nome de exemplo",
        details: [
          {
            desc: "Oferta",
            obs: "Oferta local",
            value: 50,
          },
        ],
        desc: "Depósito conta da igreja",
      },
      {
        name: "Outro nome de exemplo",
        details: [
          {
            desc: "Dízimo",
            value: 50,
          },
          {
            desc: "Oferta",
            value: 50,
          },
        ],
        desc: "Depósito conta da igreja",
      },
    ];
    const blob = new Blob([JSON.stringify(exampleJson, null, 2)], {
      type: "application/json",
    });
    const url = URL.createObjectURL(blob);
    const link = document.createElement("a");
    link.download = "exemplo.json";
    link.href = url;
    link.click();
    URL.revokeObjectURL(url);
  }

  $: if (files) {
    const read = new FileReader();
    read.readAsText(files[0]);
    read.onloadend = () => {
      setItems(JSON.parse(String(read?.result) || ""));
    };
  }
</script>

<header class="noprint">
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
  <button on:click={() => downloadExample()}>
    Baixar arquivo de exemplo
  </button>
</header>
{#each pages as page, i}
  <div class="noprint">
    <button on:click={() => printPage(i)}>{`Imprimir página ${i + 1}`}</button>
  </div>
  <div
    class={`container page ${page.print ? "" : "noprint"}`}
    id={`page${i + 1}`}
  >
    {#each page.items as item, i}
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
  header {
    font-family: sans-serif;
    padding: 10px 0px;
    font-size: 1.6em;
  }
  .container {
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
  .page {
    background-color: #fff;
    border: 1px solid;
  }
  @media print {
    .noprint {
      display: none;
    }
    .page {
      border: none;
    }
    .container {
      width: 21cm;
      height: 29.7cm;
      break-after: always;
      /* for firefox */
      page-break-after: always;
      /* for webkit */
      -webkit-column-break-after: always;
    }
  }
</style>
