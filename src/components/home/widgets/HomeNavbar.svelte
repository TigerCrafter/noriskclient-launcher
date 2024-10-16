<script>
  import { branches } from "../../../stores/branchesStore.js";
  import { defaultUser } from "../../../stores/credentialsStore.js";
  import { get } from "svelte/store";
  import { push } from "svelte-spa-router";
  import { appWindow } from "@tauri-apps/api/window";
  import { onMount } from "svelte";
  import { invoke } from "@tauri-apps/api";
  import { getNoRiskToken, noriskLog, getFeatureWhitelist, featureWhitelist, getNoRiskUser } from "../../../utils/noriskUtils.js";
  import { openInputPopup } from "../../../utils/popupUtils.js";
  import { addNotification } from "../../../stores/notificationStore.js";

  let friendInviteSlots = {};

  let navItems = [];

  function updateNavItems() {
    navItems = [
      {
        name: "PROFILES",
        onClick: () => push("/profiles"),
        image: "src/images/navbar/icons/profiles.png",
        condition: () => get(branches).length > 0 && get(defaultUser) != null,
      },
      {
        name: "SERVERS",
        onClick: () => push("/servers"),
        image: "src/images/navbar/icons/servers.png",
        condition: () => get(branches).length > 0 && get(defaultUser) != null,
      },
      {
        name: "ADDONS",
        onClick: () => push("/addons"),
        image: "src/images/navbar/icons/addons.png",
        condition: () => get(branches).length > 0 && get(defaultUser) != null,
      },
      {
        name: "INVITE",
        onClick: openInviteFriendsPopup,
        image: "src/images/navbar/icons/invite.png",
        condition: () => get(branches).length > 0 && get(defaultUser) != null && $featureWhitelist.includes("INVITE_FRIENDS") && friendInviteSlots.availableSlots == -1,
      },
      {
        name: "CAPES",
        onClick: () => push("/capes"),
        image: "src/images/navbar/icons/capes.png",
        condition: () => get(branches).length > 0 && get(defaultUser) != null,
      },
      {
        name: "SKIN",
        onClick: () => push("/skin"),
        image: "src/images/navbar/icons/skin.png",
        condition: () => get(defaultUser) != null,
      },
      {
        name: "SETTINGS",
        onClick: () => push("/launcher-settings"),
        className:  "settings",
        image: "src/images/navbar/icons/settings.png",
        condition: true
      },
    ];
  }

  let showNavBar = false;

  function toggleNavItems() {
    showNavBar = !showNavBar
  }
  
  
  onMount(async () => {
    const branchesUnlisten = branches.subscribe(async value => {
      updateNavItems();
    });

    const userUnlisten = defaultUser.subscribe(async value => {
      updateNavItems();
      await fetchFeatures();
      await getNoRiskUser();
      updateNavItems();
    });

    return () => {
      branchesUnlisten();
      userUnlisten();
    };
  });

  async function fetchFeatures() {
    await getFeatureWhitelist();

    if ($featureWhitelist.includes("INVITE_FRIENDS")) {
      await loadFriendInvites();
    }
  }

  async function loadFriendInvites() {
    if (!$defaultUser) return;
    await invoke("get_whitelist_slots", {
      noriskToken: getNoRiskToken(),
      uuid: $defaultUser.id,
    }).then((result) => {
      noriskLog("Received Whitelist Slots" + JSON.stringify(result));
      friendInviteSlots = result;
      friendInviteSlots.text = friendInviteSlots.availableSlots === -1 ? '∞' : `${friendInviteSlots.availableSlots - friendInviteSlots.previousInvites}/${friendInviteSlots.availableSlots}`;
    }).catch((reason) => {
      addNotification(reason);
      friendInviteSlots = {};
    });
  }

  function openInviteFriendsPopup() {
    openInputPopup({
      title: "Invite Friends",
      content: `You have ${friendInviteSlots.text} invites left.\nYou can use them to invite a friend to the NRC closed beta.`,
      inputPlaceholder: "Username / UUID",
      confirmButton: "Invite",
      validateInput: (input) => input.length > 2 && input.length <= 16,
      onConfirm: async (identifier) => {        
        await invoke("add_player_to_whitelist", {
          identifier: identifier,
          noriskToken: getNoRiskToken(),
          requestUuid: $defaultUser.id,
        }).then(() => {
          addNotification("Successfully invited " + identifier + " to the NRC closed beta!", "INFO");
        }).catch((error) => {
          addNotification("An error occurred while inviting " + identifier + " to the NRC closed beta: " + error);
        });
      }
    })
  }
</script>


<div class="container">
  <div class="topleft">
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <h1 class="toggle-button" on:click={toggleNavItems}>
      {showNavBar ? 'MENU ←' : 'MENU →'}
    </h1>
    <div class="home-navbar-wrapper" style="transform: {showNavBar ? 'translateX(0)' : 'translateX(-100%)'};">
      {#if $featureWhitelist.includes("INVITE_FRIENDS") && friendInviteSlots.availableSlots !== -1 && friendInviteSlots.availableSlots - friendInviteSlots.previousInvites > 0}
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <h1 class="invite-button" on:click={openInviteFriendsPopup}>
          <p>✨ INVITE ✨</p>
        </h1>
      {/if}
        {#each navItems as item (item.name)}
          {#if typeof item.condition === 'function' ? item.condition() : item.condition}
          <!-- svelte-ignore a11y-click-events-have-key-events -->
            <div  class={item.className || ''} on:click={item.onClick}>
              {#if item.image && item.image !== ''}
                <img src={item.image} alt={item.name} />
              {/if}
              <h1>
                {item.name}
              </h1>
            </div>
          {/if}
        {/each}
    </div>
  </div>
</div>

<style>
  .container {
    position: absolute;
    width: 720px;
    height: 80vh;
    pointer-events: none;
  }

  .topleft {
    position: absolute;
    top: 0;
    left: 0;
    bottom: 0;
  }

  .toggle-button {
    pointer-events: all;
    font-size: 11.5px;
    font-family: 'Press Start 2P', serif;
    margin-bottom: 1em;
    cursor: pointer;
    color: var(--secondary-color);
    text-shadow: 1px 1px var(--secondary-color-text-shadow);
    transition: transform 0.15s, color 0.25s, text-shadow 0.25s;
  }

  .home-navbar-wrapper {
    width: 200%;
    bottom: 0;
    top: 2.5%;
    position: absolute;
    padding: 10px;
    display: flex;
    flex-direction: column;
    align-items: start;
    pointer-events: all;
    overflow: hidden;
    transition: transform 1s;
    border-top-right-radius: 10px;
    border-bottom-right-radius: 10px;
    background-color: rgb(0, 145, 255);
  }

  .home-navbar-wrapper div {
  display: flex;
  align-items: center;
  margin-bottom: 1em;
  }

  .home-navbar-wrapper h1 {
    font-size: 11.5px;
    font-family: 'Press Start 2P', serif;
    margin-bottom: 1em;
    cursor: pointer;
    color: var(--secondary-color);
    text-shadow: 1px 1px var(--secondary-color-text-shadow);
    transition: transform 0.15s, color 0.25s, text-shadow 0.25s;
  }

  .home-navbar-wrapper img {
    max-height: 16px;
    margin-bottom: 10px;
    margin-right: 5px;
    transition: transform 0.15s;
  }
  
  .home-navbar-wrapper img:hover {
    transform: scale(1.2);
  }

  .home-navbar-wrapper h1:hover {
    color: var(--hover-color);
    text-shadow: 1px 1px var(--hover-color-text-shadow);
    transform: scale(1.1) translateX(8px) perspective(1px);
  }

  .home-navbar-wrapper div.settings {
    position: absolute;
    bottom: 0;
  }

  .home-navbar-wrapper h1.invite-button {
    display: flex;
    flex-direction: row;
    align-items: center;
    font-size: 12.5px;
  }

  .home-navbar-wrapper h1.invite-button p {
    margin-bottom: 5px;
    padding-right: 5px;
    font-size: 15px;
  }
</style>
