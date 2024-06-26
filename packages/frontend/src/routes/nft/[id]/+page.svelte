<script>
  /** @type {import('./$types').PageData} */
  import { Button } from "$lib/components/ui/button";
  import * as Card from "$lib/components/ui/card";
  import { GetUSDExchangeRate } from "$lib/utils.js";
  import { onMount } from "svelte";

  import {
    defaultEvmStores as evm,
    contracts,
    selectedAccount,
  } from "svelte-web3";
  import {
    getFileFromIPFS,
    CONTRACT_ABI,
    CONTRACT_ADDRESS,
  } from "$lib/utils.js";
  export let data;
  let nft = data.nftData.nft;
  // Reactive variable for USD price
  let usdPrice = "";
  onMount(async () => {
    await evm.setProvider();
    await evm.attachContract("Clipper", CONTRACT_ADDRESS, CONTRACT_ABI);
    await fetchNFTData();
    fetchUSDPrice();
  });

  // Function to fetch and set the USD price
  const fetchUSDPrice = async () => {
    usdPrice = "$" + (await GetUSDExchangeRate(nft.price));
  };
  // Fetch USD price on component mount
  async function fetchNFTData() {
    let id = data.nftData.id;
    const info = await $contracts.Clipper.methods
      .tokenInfo(id)
      .call({ from: $selectedAccount });
    console.log(info);
    nft = {
      id: id,
      name: info.name,
      img: getFileFromIPFS(info.image_cid),
      price: info.price,
      saleEnd: info.saleEnd,
      description: info.description,
    };
  }
</script>

{#if nft}
  <div class="container mx-auto p-4">
    <div class="flex flex-col lg:flex-row lg:space-x-8">
      <!-- NFT Image Section -->
      <div class="flex-shrink-0 w-full lg:w-1/2">
        <Card.Root class="rounded-lg shadow-lg">
          <img src={nft.img} alt="NFT" class="nft-image rounded-lg" />
        </Card.Root>
      </div>

      <!-- NFT Details Section -->
      <div class="flex-grow mt-4 lg:mt-0 w-full lg:w-1/2 space-y-4">
        <div class="text-3xl font-bold">{nft.name}</div>
        <div class="text-lg text-gray-400">#{nft.id}</div>
        <div class="text-gray-500">
          Owned by
          {#await $contracts.Clipper.methods.ownerOf(nft.id).call()}
            ...
          {:then owner}
            <span class="text-blue-500">
              {owner}
            </span>
          {/await}
        </div>
        <div class="text-xl font-bold flex items-center space-x-2">
          <span>{nft.price} ETH</span>
          <span class="text-sm text-gray-500">{usdPrice}</span>
        </div>
        <div class="flex space-x-2">
          <Button
            variant="primary"
            class="w-full lg:w-1/2 bg-blue-500 text-white rounded hover:bg-blue-600 transition"
          >
            Buy now
          </Button>
          <Button variant="outline" size="lg" class="w-full lg:w-1/2">
            Make offer
          </Button>
        </div>
        <div class="text-gray-400">Sale ends {nft.saleEnd}</div>

        <!-- Support creator info -->
        <div class="text-sm text-red-500">Supports creator</div>

        <!-- Description Section -->
        <div class="mt-6">
          <h3 class="text-lg font-semibold">Description</h3>
          <p class="text-gray-400">{nft.description}</p>
        </div>
      </div>
    </div>
  </div>
{/if}

<style>
  .container {
    background-color: #1a1a1a;
    color: #f1f1f1;
  }
  .nft-image {
    width: 700px;
    height: 700px;
    object-fit: cover;
    border-bottom: 1px solid #333;
  }
  /* .button-wide {
    width: 100%;
  } */
  .text-lg {
    font-size: 1.125rem;
  }
  .text-xl {
    font-size: 1.25rem;
  }
  /* .text-2xl {
    font-size: 1.5rem;
  } */
  .text-3xl {
    font-size: 1.875rem;
  }
</style>
