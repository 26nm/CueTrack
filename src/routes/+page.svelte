<script lang="ts">
    type Session = {
        id: number;
        date: string;
        location: string;
        gameType: string;
        sessionType: string;
        wins: number;
        losses: number;
        notes: string;
    };

    let sessions = $state<Session[]>([]);

    let date = $state('');
    let location = $state('');
    let gameType = $state('8-ball');
    let sessionType = $state('Practice');
    let wins = $state(0);
    let losses =$state(0);
    let notes = $state('')

    function addSession() {
        const newSession: Session = {
            id: Date.now(),
            date,
            location,
            gameType,
            sessionType,
            wins,
            losses,
            notes
        };

        sessions = [newSession, ...sessions];

        date = '';
        location = '';
        gameType = '8-ball';
        sessionType = 'Practice';
        wins = 0;
        losses = 0;
        notes = '';
    }
</script>

<h1>CueTrack</h1>
<p>Track pool sessions, match results, and practice notes.</p>

<form 
    onsubmit={(event) => {
        event.preventDefault();
        addSession();
    }}
>
    <label>
        Date
        <input type="date" bind:value={date} required />
    </label>

    <label>
        Location
        <input type="text" bind:value={location} placeholder="e.g. Ox Billiards" required />
    </label>

    <label>
        Game Type
        <select bind:value={gameType}>
            <option>8-ball</option>
            <option>9-ball</option>
            <option>10-ball</option>
            <option>Straight Pool</option>
            <option>Drills</option>
        </select>
    </label>

    <label>
        Wins
        <input type="number" min="0" bind:value={wins} />
    </label>

    <label>
        Losses
        <input type="number" min="0" bind:value={losses} />
    </label>

    <label>
        Notes
        <textarea bind:value={notes} placeholder="What did you notice today?"></textarea>
    </label>

    <button type="submit">Add Session</button>
</form>

<section>
    <h2>Session History</h2>

    {#if sessions.length === 0}
        <p>No sessions logged yet.</p>
    {:else}
        {#each sessions as session}
            <article>
                <h3>{session.gameType} at {session.location}</h3>
                <p>{session.date} · {session.sessionType}</p>
                <p>Record: {session.wins}W - {session.losses}L</p>
                <p>{session.notes}</p>
            </article>
        {/each}
    {/if}
</section>
