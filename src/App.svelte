<script>
  let user = {
    name: "Sarah M.",
    startDate: new Date("2025-09-01"),
    daysActive: 7,
    streak: 5
  }

  let mood = ""
  let sleep = ""
  let stress = 50
  let gratitude = ""
  let saved = false
  let editing = false
  let editIndex = null

  //some mock entries
  let history = [
    { date: "9/20/2025", mood: "😀 Happy", sleep: 7, stress: 40, gratitude: "Had coffee with a friend" },
    { date: "9/21/2025", mood: "😢 Sad", sleep: 5, stress: 70, gratitude: "Read a comforting book" }
  ]

  function saveEntry() {
    if (editing && editIndex !== null) {
      history[editIndex] = {
        ...history[editIndex],
        date: history[editIndex].date + " (previous)"
      }
      
      history.push({
        date: history[editIndex].date.replace(" (previous)", "") + " (updated)",
        mood, sleep, stress, gratitude
      })
      saved = true
      editing = false
      editIndex = null
    } else {
      saved = true
      user.streak += 1
      history.push({
        date: new Date().toLocaleDateString(),
        mood, sleep, stress, gratitude
      })
    }
    resetForm()
  }

  function cancelEntry() {
    resetForm()
    editing = false
    editIndex = null
    saved = false
  }

  function resetForm() {
    mood = ""
    sleep = ""
    stress = 50
    gratitude = ""
  }

  function editEntry(index) {
    let entry = history[index]
    mood = entry.mood
    sleep = entry.sleep
    stress = entry.stress
    gratitude = entry.gratitude
    editing = true
    editIndex = index
    saved = false
  }

  function daysSinceStart() {
    const today = new Date()
    const diff = today - user.startDate
    return Math.floor(diff / (1000 * 60 * 60 * 24))
  }

  $: stressLabel =
    stress <= 25 ? "Low" :
    stress <= 50 ? "Medium" :
    stress <= 75 ? "High" : "Very High"

  $: stressColor =
    stress <= 25 ? "#3b82f6" :
    stress <= 50 ? "#0ea5e9" :
    stress <= 75 ? "#f97316" :
                   "#dc2626"

  //calendar
  let today = new Date()
  let year = today.getFullYear()
  let month = today.getMonth()
  let daysInMonth = new Date(year, month + 1, 0).getDate()

  function formatDate(y, m, d) {
    return `${m+1}/${d}/${y}`
  }

  function findEntryByDate(dateStr) {
    return history.findIndex(e => e.date.startsWith(dateStr))
  }

  //level 3
  let goalSleep = 7
  let goalStress = 50
  let trackMood = true
  let trackSleep = true
  let trackStress = true
  let trackGratitude = true

  $: avgSleep = history.length > 0 ? Math.round(history.reduce((a,e)=>a+Number(e.sleep||0),0)/history.length) : 0
  $: avgStress = history.length > 0 ? Math.round(history.reduce((a,e)=>a+Number(e.stress||0),0)/history.length) : 0

  // level 4
let reminderOn = false
let reminderMessage = ""

</script>

<main>
  <h1>Daily Journal</h1>

  <section class="dashboard">
    <h2>Dashboard Overview</h2>
    <p><strong>Name:</strong> {user.name}</p>
    <p><strong>Days since start:</strong> {daysSinceStart()}</p>
    <p><strong>Days active:</strong> {user.daysActive}</p>
    <p><strong>Streak:</strong> {user.streak} days</p>
    {#if history.length > 0}
      <p><strong>Average Sleep:</strong> {avgSleep} hrs</p>
      <p><strong>Average Stress:</strong> {avgStress}</p>
    {/if}
  </section>

  <!-- level 3 -->
  <section class="goals">
    <h2>Set Goals</h2>

    <div>
      <label for="goalSleep">Sleep Goal (hrs):</label>
      <input id="goalSleep" type="number" bind:value={goalSleep} min="0" max="24" />
    </div>

    <div>
      <label for="goalStress">Stress Goal (0–100):</label>
      <input id="goalStress" type="number" bind:value={goalStress} min="0" max="100" />
    </div>
  </section>

  <section class="form">
    <h2>{editing ? "Edit Entry" : "Log Today"}</h2>

    {#if trackMood}
      <label for="mood">Mood:</label>
      <select id="mood" bind:value={mood}>
        <option value="">Select...</option>
        <option>😀 Happy</option>
        <option>😢 Sad</option>
        <option>😠 Angry</option>
        <option>😨 Fearful</option>
        <option>🤢 Disgusted</option>
        <option>😲 Surprised</option>
        <option>😒 Contempt</option>
      </select>
    {/if}

    {#if trackSleep}
      <label for="sleep">Sleep (hours):</label>
      <input id="sleep" type="number" bind:value={sleep} min="0" max="24" />
    {/if}

    {#if trackStress}
      <label for="stress">Stress:</label>
      <input id="stress" type="range" min="0" max="100" bind:value={stress} />
      <p class="stress-label" style="color:{stressColor}">
        Stress level: {stressLabel} ({stress})
      </p>
    {/if}

    {#if trackGratitude}
      <label for="gratitude">Gratitude (optional):</label>
      <textarea id="gratitude" bind:value={gratitude}></textarea>
    {/if}

    <div class="buttons">
      <button class="save" on:click={saveEntry}>{editing ? "Update" : "Save"}</button>
      <button class="cancel" on:click={cancelEntry}>Cancel</button>
    </div>

    {#if saved}
      <p class="feedback">{editing ? "Entry updated successfully" : "Entry saved successfully"}</p>
    {/if}
  </section>

  <!-- level 3 -->
  <section class="overview">
    <h2>Performance Overview</h2>
    <p>Goal Sleep: {goalSleep} hrs | Average Sleep: {avgSleep} hrs</p>
    <p>Goal Stress: {goalStress} | Average Stress: {avgStress}</p>
    <p style="color:{avgSleep >= goalSleep ? 'green' : 'red'}">{avgSleep >= goalSleep ? "Sleep goal met" : "Sleep goal not met"}</p>
    <p style="color:{avgStress <= goalStress ? 'green' : 'red'}">{avgStress <= goalStress ? "Stress goal met" : "Stress goal not met"}</p>
  </section>

  <!-- level 3 -->
  <section class="customize">
    <h2 id="customizeTitle">Customize Tracking</h2>

    <div>
      <input id="trackMood" type="checkbox" bind:checked={trackMood} />
      <label for="trackMood">Track Mood</label>
    </div>

    <div>
      <input id="trackSleep" type="checkbox" bind:checked={trackSleep} />
      <label for="trackSleep">Track Sleep</label>
    </div>

    <div>
      <input id="trackStress" type="checkbox" bind:checked={trackStress} />
      <label for="trackStress">Track Stress</label>
    </div>

    <div>
      <input id="trackGratitude" type="checkbox" bind:checked={trackGratitude} />
      <label for="trackGratitude">Track Gratitude</label>
    </div>
  </section>

  <!-- level 4 -->
  <section class="reminder">
    <h2>Reminders</h2>
    <label for="reminderToggle">
      <input id="reminderToggle" type="checkbox" bind:checked={reminderOn} 
        on:change={() => reminderMessage = reminderOn ? "Daily reminder set for 8 PM" : "Reminder turned off"} />
      Enable daily reminder at 8 PM
    </label>
    {#if reminderMessage}
      <p class="feedback">{reminderMessage}</p>
    {/if}
  </section>



  <section class="calendar">
    <h2>History</h2>
    {#if history.length === 0}
      <p>No entries yet.</p>
    {:else}
      <ul>
        {#each history as entry, i}
          <li>
            <button class="history-btn" on:click={() => editEntry(i)}>
              <strong>{entry.date}:</strong>
              Mood {entry.mood || "-"},
              Sleep {entry.sleep || "-"} hrs,
              Stress {entry.stress},
              Gratitude: {entry.gratitude || "-"}
            </button>
          </li>
        {/each}
      </ul>
      <p>(Click an entry above to edit)</p>
    {/if}
  </section>

  <section class="calendar">
    <h2>Calendar View</h2>
    <div class="calendar-grid">
      {#each Array(daysInMonth) as _, day}
        {#if findEntryByDate(formatDate(year, month, day+1)) !== -1}
          <button
            class="day has-entry"
            on:click={() => editEntry(findEntryByDate(formatDate(year, month, day+1)))}>
            {day+1}
          </button>
        {:else}
          <div class="day">{day+1}</div>
        {/if}
      {/each}
    </div>
    <p>(Click a highlighted day to edit)</p>
  </section>
</main>

<style>
  :global(body) {
    margin: 0;
    padding: 0;
    background: #e0f2f6;
    font-family: Arial, sans-serif;
  }

  main {
    max-width: 600px;
    margin: 2rem auto;
    padding: 2rem;
    background: #ffffff;
    color: #1e293b;
    border-radius: 10px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  }

  h1 {
    text-align: center;
    font-size: 1.8rem;
    margin-bottom: 1.5rem;
    color: #0ea5e9;
  }

  h2 {
    font-size: 1.2rem;
    margin-bottom: 0.6rem;
    border-bottom: 1px solid #cbd5e1;
    padding-bottom: 0.3rem;
    color: #0f172a;
  }

  section {
    margin-bottom: 2rem;
    background: #f9fafb;
    padding: 1rem;
    border-radius: 8px;
  }

  label {
    display: block;
    margin-top: 1rem;
    font-weight: bold;
  }

  input,
  select,
  textarea {
    width: 100%;
    margin-top: 0.3rem;
    padding: 0.6rem;
    border-radius: 6px;
    border: 1px solid #cbd5e1;
    font-size: 0.95rem;
    background: #ffffff;
    color: #1e293b;
    box-sizing: border-box;
  }

  textarea {
    resize: vertical;
    min-height: 60px;
  }

  input[type="range"] {
    -webkit-appearance: none;
    width: 100%;
    height: 6px;
    border-radius: 4px;
    background: #cbd5e1;
    outline: none;
  }

  input[type="range"]::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    width: 18px;
    height: 18px;
    border-radius: 50%;
    background: #3b82f6;
    cursor: pointer;
    border: 2px solid #ffffff;
  }

  .buttons {
    margin-top: 1rem;
    display: flex;
    gap: 0.6rem;
  }

  button {
    flex: 1;
    padding: 0.6rem 1.2rem;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    font-weight: bold;
    transition: background 0.2s ease;
  }

  .save {
    background: #34d399;
    color: white;
  }

  .cancel {
    background: #f87171;
    color: white;
  }

  .feedback {
    margin-top: 0.8rem;
    color: #34d399;
    font-weight: bold;
  }

  .stress-label {
    margin-top: 0.4rem;
    font-weight: bold;
  }

  .history-btn {
    all: unset;
    cursor: pointer;
    display: block;
    width: 100%;
    text-align: left;
    padding: 0.3rem;
    border-radius: 4px;
  }

  .history-btn:hover {
    background: #f1f5f9;
  }

  .calendar-grid {
    display: grid;
    grid-template-columns: repeat(7, 1fr);
    gap: 0.3rem;
    margin-top: 1rem;
  }

  .day {
    padding: 0.5rem;
    text-align: center;
    border-radius: 4px;
    background: #f9fafb;
    border: 1px solid #e2e8f0;
  }

  .has-entry {
    background: #34d399;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }

  .has-entry:hover {
    background: #059669;
  }

  .reminder {
  margin-bottom: 2rem;
  background: #f9fafb;
  padding: 1rem;
  border-radius: 8px;
}

.reminder label {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-weight: bold;
}
</style>