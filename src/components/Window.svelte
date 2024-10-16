<script>
  import { appWindow } from "@tauri-apps/api/window";
</script>

<div>
  <!-- svelte-ignore a11y-click-events-have-key-events -->
  <h1 class="window" id="quit" on:click={() => appWindow.close()}>X</h1>
  <!-- svelte-ignore a11y-click-events-have-key-events -->
  <h1 class="window" id="minimize" on:click={() => appWindow.minimize()}>-</h1>
</div>

<style>
  .window{
    padding: 10px;
    cursor: pointer;
    font-size: 25px;
    transition: transform 0.2s ease-in-out;
    text-align: center;
    border-radius: 5px;
    display: flex;
    justify-content: center;
    align-items: center;
    box-shadow: 2px 2px 5px rgba(0, 0, 0, 0.5);
    cursor: pointer;
    top: 20px;
    right: 0px;
    position: absolute;
  }

  div {
    position: absolute;
    right: 0px;
    top: 0px;
    background-color: grey;
    padding-left: 140px;
    padding-bottom: 65px;
    border-radius: 0.75em;
  }

  #quit:hover, #minimize:hover {
    transform: scale(1.2);
  }
  
  #quit {
    color: red;
    margin-right: 20px;
    scale: 1.5;
    width: 32px;
    height: 32px;
    margin-top: -3px;
  }
  #minimize {
    color: green;
    margin-right: 90px;
    scale: 2;
    width: 25px;
    height: 25px;
  }
</style>