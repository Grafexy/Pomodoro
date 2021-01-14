<script>
  import { onDestroy, onMount } from "svelte";
  import dayjs from "dayjs";
  import dayjsDuration from "dayjs/plugin/duration";
  dayjs.extend(dayjsDuration);

  let inputs = [
    {
      text: "Infinity",
      timesFinished: 0,
      elapsed: 0,
    },
    {
      text: "Cultural Fit",
      timesFinished: 0,
      elapsed: 0,
    },
    {
      text: "Hacker Rank",
      timesFinished: 0,
      elapsed: 0,
    },
  ];

  let mounted = false;

  $: if (mounted) {
    localStorage.setItem("inputs", JSON.stringify(inputs));
  }

  function addProject() {
    inputs.push({
      text: "",
      timesFinished: 0,
      elapsed: 0,
    });
    inputs = inputs;
  }

  function removeProject(i) {
    inputs.splice(i, 1);
    inputs = inputs;
    if (activeIndex && activeIndex === i) {
      activeIndex--;
    }
    if (!inputs.length) {
      addProject();
    }
  }
  function addTimes(i) {
    inputs[i].timesFinished += 1;
    inputs = inputs;
  }
  function substructTimes(i) {
    inputs[i].timesFinished -= 1;
    inputs = inputs;
  }

  const rf = (callback) => {
    return setTimeout(function () {
      callback(true);
    }, 16);
  };

  let activeIndex = 0;

  $: selectedInput = inputs[activeIndex];

  function changeActive(i) {
    activeIndex = i;
  }

  let duration = 25 * 60 * 1000;

  let frame;
  let running = false;

  function startTimer(i) {
    Notification.requestPermission();
    let last_time = window.performance.now();
    if (i === activeIndex && running) return;
    pauseTimer();
    activeIndex = typeof i === "number" ? i : activeIndex;
    running = true;
    (function update() {
      frame = rf(update);

      const time = window.performance.now();
      inputs[activeIndex].elapsed += Math.min(
        time - last_time,
        duration - inputs[activeIndex].elapsed
      );
      inputs = inputs;
      last_time = time;
      if (inputs[activeIndex].elapsed >= duration) {
        endTimeout(true);
      }
    })();
  }

  function pauseTimer() {
    clearTimeout(frame);
    running = false;
  }

  function cancelTimer() {
    selectedInput.elapsed = 0;
    inputs = inputs;
    pauseTimer();
  }

  function endTimeout(finished) {
    selectedInput.timesFinished += 1;
    cancelTimer();
    if (finished) {
      const notification = new Notification("Time finished", {
        body: selectedInput.elapsed,
      });
    }
  }

  function resetTimesFinished(i) {
    inputs[i].timesFinished = 0;
    inputs = inputs;
  }

  function resetTime(i) {
    inputs[i].elapsed = 0;
    inputs = inputs;
  }

  onMount(() => {
    const oldInputs = localStorage.getItem("inputs");
    if (oldInputs) {
      inputs = JSON.parse(oldInputs);
    }
    mounted = true;
  });

  onDestroy(() => {
    clearTimeout(frame);
  });
</script>

<svelte:head>
  <title>
    {running
      ? `${dayjs
          .duration(25 * 60 * 1000 - selectedInput.elapsed)
          .format("HH:mm:ss")} - ${selectedInput.text}`
      : "Pomodoro"}
  </title>
</svelte:head>
<main>
  <section>
    <table>
      <thead>
        <tr>
          <th />
          <th>Project</th>
          <th>Times finished</th>
          <th>Current time</th>
        </tr>
      </thead>
      <tbody>
        {#each inputs as input, i}
          <tr>
            <td>
              <button on:click={() => changeActive(i)}>Select</button>
            </td>
            <td>
              <input type="text" bind:value={input.text} />
            </td>
            <td>
              <button class="icon" on:click={() => addTimes(i)}>+</button>
              {input.timesFinished}
              <button class="icon" on:click={() => substructTimes(i)}>-</button>
            </td>
            <td>
              {dayjs.duration(input.elapsed).format("HH:mm:ss")}
            </td>
            <td>
              <button class="icon" on:click={() => removeProject(i)}>🗙</button>
              <button on:click={() => resetTimesFinished(i)}
                >Clear Finished</button
              >
              <button on:click={() => resetTime(i)}>Clear Time</button>
            </td>
          </tr>
        {/each}
      </tbody>
      <tfoot>
        <tr>
          <td>
            <button on:click={addProject}>Add new project</button>
          </td>
        </tr>
      </tfoot>
    </table>
  </section>
  <section>
    <h2>{selectedInput.text}</h2>
    <h1>
      {dayjs
        .duration(25 * 60 * 1000 - selectedInput.elapsed)
        .format("HH:mm:ss")}
    </h1>
    <div class="inp">
      <button on:click={startTimer}>▶️</button>
      <button on:click={pauseTimer}>⏸️</button>
      <button on:click={cancelTimer}>⏹️</button>
      <button on:click={endTimeout}>✔️</button>
    </div>
  </section>
</main>

<style>
  h1 {
    font-size: 4em;
    margin-top: 0.25em;
  }
  h2 {
    font-size: 3em;
    margin-bottom: 0.25em;
    color: #999;
  }
  main {
    display: flex;
    width: 100vw;
    height: 100%;
  }

  button {
    border: 1px solid #999;
    background: none;
    border-radius: 4px;
    margin-bottom: 0;
  }
  button:hover {
    border: 1px solid rgb(175, 83, 83) !important;
  }
  section {
    flex: 1;
    display: flex;
    flex-flow: column;
    justify-content: center;
    align-items: center;
  }
  .inp {
    display: flex;
  }
  .inp button {
    border-color: transparent;
  }
  th,
  td {
    text-align: center;
    padding: 4px 8px;
  }
  .icon {
    border-radius: 50%;
    border-color: transparent;
    background-color: #eee;
    text-align: center;
    /* padding: 0.5em; */
    width: 4ch;
    height: 4ch;
    padding-bottom: 0.2em;
    padding-top: 0;
  }
</style>
