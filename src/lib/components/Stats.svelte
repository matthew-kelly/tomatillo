<script>
  export let timer;
</script>

<div class="container">
  <h1>Stats</h1>
  <p>Pomodoros completed: {timer.sessions}</p>
</div>

<style>
  .container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    margin: 1rem;
  }

  h1 {
    margin-bottom: 16px;
  }

  p {
    margin-bottom: 4px;
  }
</style>
