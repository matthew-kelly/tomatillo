<script>
  import { onMount } from "svelte";
  import { writable } from "svelte/store";
  import Modal, { bind } from "svelte-simple-modal";
  import Stats from "./Stats.svelte";
  import { sendNotification } from "@tauri-apps/api/notification";

  const modal = writable(null);
  function showModal() {
    modal.set(bind(Stats, { timer: $timer }));
  }

  const minutes = {
    pomodoro: 25,
    shortBreak: 5,
    longBreak: 15,
  };

  const mode = writable("pomodoro"); // pomodoro, shortBreak, longBreak
  const timer = writable({
    total: minutes[$mode] * 60,
    minutes: minutes[$mode],
    seconds: 0,
    longBreakInterval: 4,
    sessions: 0,
  });

  $: clockMinutes = $timer.minutes < 10 ? `0${$timer.minutes}` : $timer.minutes;
  $: clockSeconds = $timer.seconds < 10 ? `0${$timer.seconds}` : $timer.seconds;
  $: progress = $timer.total - ($timer.minutes * 60 + $timer.seconds);

  let interval;
  let isRunning = false;

  const alarm = new Audio("/sounds/alarm.mp3");

  function startTimer() {
    isRunning = true;

    interval = setInterval(() => {
      if ($timer.seconds > 0) {
        $timer.seconds--;
      } else {
        if ($timer.minutes > 0) {
          $timer.minutes--;
          $timer.seconds = 59;
        } else {
          alarm.play();
          notify();
          switch ($mode) {
            case "pomodoro":
              $timer.sessions += 1;
              if ($timer.sessions % $timer.longBreakInterval === 0) {
                setMode("longBreak");
              } else {
                setMode("shortBreak");
              }
              break;
            default:
              setMode("pomodoro");
          }
          startTimer();
        }
      }
    }, 1000);
  }

  function stopTimer() {
    isRunning = false;
    clearInterval(interval);
  }

  function setMode(newMode) {
    stopTimer();
    $mode = newMode;
    $timer.total = minutes[$mode] * 60;
    $timer.minutes = minutes[$mode];
    $timer.seconds = 0;
  }

  function notify() {
    let message;
    switch ($mode) {
      case "pomodoro":
        message = "Pomodoro completed! Time for a break.";
        break;
      case "shortBreak":
        message = "Short break completed! Back to work!";
        break;
      case "longBreak":
        message = "Long break completed! Back to work!";
        break;
    }
    const options = {
      title: "Tomatillo",
      body: message,
      icon: "/images/tomatillo.png",
    };
    sendNotification(options);
  }

  onMount(() => {
    setMode("pomodoro");
  });
</script>

<Modal show={$modal} />

<div class="timer-container" style="background-color: var(--{$mode})">
  <progress value={progress} max={$timer.total} />
  <div class="progress-bar" />
  <div class="timer">
    <div class="button-group mode-buttons">
      <button
        type="button"
        class="button mode-button"
        class:active={$mode === "pomodoro"}
        on:click={() => setMode("pomodoro")}
      >
        Pomodoro
      </button>
      <button
        type="button"
        class="button mode-button"
        class:active={$mode === "shortBreak"}
        on:click={() => setMode("shortBreak")}
      >
        Short Break
      </button>
      <button
        type="button"
        class="button mode-button"
        class:active={$mode === "longBreak"}
        on:click={() => setMode("longBreak")}
      >
        Long Break
      </button>
    </div>

    <div class="clock">
      {clockMinutes}:{clockSeconds}
    </div>

    <div class="button-group">
      <button
        type="button"
        class="main-button"
        class:active={isRunning}
        on:click={() => {
          if (isRunning) stopTimer();
          else startTimer();
        }}
      >
        {isRunning ? "Stop" : "Start"}
      </button>
      <button type="button" class="main-button" on:click={() => setMode($mode)}>
        Reset
      </button>
    </div>
  </div>
  <button type="button" class="mode-button active" on:click={showModal}>
    Stats
  </button>
</div>

<style>
  :root {
    --pomodoro: hsl(4, 82%, 56%);
    --shortBreak: hsl(200, 63%, 56%);
    --longBreak: #26a65b;
  }

  .timer-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100vh;
    padding-bottom: 1rem;
    transition: background-color 0.5s ease;
  }

  progress {
    border-radius: 2px;
    width: 100%;
    height: 12px;
    position: fixed;
    top: 0;
  }

  progress::-webkit-progress-bar,
  progress::-moz-progress-bar {
    background-color: rgba(0, 0, 0, 0.1);
  }
  progress::-webkit-progress-value {
    background-color: rgba(255, 253, 197, 0.75);
  }

  .timer {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    flex-grow: 1;
    width: 100%;
    padding: 20px;
    text-align: center;
  }

  .clock {
    margin-top: 48px;
    margin-bottom: 24px;
    font-size: 10rem;
    display: flex;
    align-items: center;
    font-family: "clockface", arial, sans-serif;
  }

  .mode-button {
    font-size: 16px;
    height: 28px;
    cursor: pointer;
    box-shadow: none;
    font-weight: 300;
    color: #fff;
    border: 1px solid transparent;
    margin: 0px;
    border-radius: 4px;
    padding: 2px 12px;
    background: none;
  }

  .mode-button.active {
    border-color: #fff;
  }

  .main-button {
    cursor: pointer;
    box-shadow: -6px 6px 0px rgba(0, 0, 0, 0.2);
    font-size: 22px;
    height: 55px;
    color: black;
    font-weight: bold;
    width: 200px;
    background-color: white;
    border-width: initial;
    border-style: none;
    margin: 20px 10px 10px;
    padding: 0px 12px;
    border-radius: 4px;
    transition: all 0.1s ease-in-out 0s;
  }

  button:focus,
  button:active {
    outline: none;
  }

  .main-button.active {
    transform: translate(-6px, 6px);
    background-color: #ddd;
    box-shadow: none;
    outline: none;
  }

  @media screen and (max-width: 550px) {
    .clock {
      font-size: 7rem;
    }
  }
</style>
