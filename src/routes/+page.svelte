<script lang="ts">
    import { onMount } from 'svelte';
    import { supabase } from '$lib/supabaseClient';

    type Session = {
        id: string;
        date: string;
        location: string;
        game_type: string;
        session_type: string;
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
    let editingSessionId = $state<string | null>(null);

    async function fetchSessions() {
        const { data, error } = await supabase
            .from('sessions')
            .select('*')
            .order('created_at', { ascending: false });

        if(error) {
            console.error(error);
            return;
        }

        sessions = data ?? [];
    }

    async function addSession() {
        const sessionPayload = {
            date,
            location,
            game_type: gameType,
            session_type: sessionType,
            wins,
            losses,
            notes
        };

        if(editingSessionId) {
            const { error } = await supabase
                .from('sessions')
                .update(sessionPayload)
                .eq('id', editingSessionId);

            if(error) {
                console.error(error);
                return;
            }

            editingSessionId = null;
            resetForm();
            await fetchSessions();
            return;
        }

        const { error } = await supabase.from('sessions').insert(sessionPayload);

        if(error) {
            console.error(error);
            return;
        }

        resetForm();
        await fetchSessions();
    }

    function resetForm() {
        date = '';
        location = '';
        gameType = '8-ball';
        sessionType = 'Practice';
        wins = 0;
        losses = 0;
        notes = '';
    }

    function startEdit(session: Session) {
        editingSessionId = session.id;
        date = session.date;
        location = session.location;
        gameType = session.game_type;
        sessionType = session.session_type;
        wins = session.wins;
        losses = session.losses;
        notes = session.notes;
    }

    async function deleteSession(id: string) {
        const { error } = await supabase.from('sessions').delete().eq('id', id);

        if(error) {
            console.error(error);
            return;
        }

        sessions = sessions.filter((session) => session.id !== id);
    }

    onMount(fetchSessions);
</script>
<main>
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
        Session Type
        <select bind:value={sessionType}>
            <option>Practice</option>
            <option>Match</option>
            <option>Tournament</option>
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

    <button type="submit">
        {editingSessionId ? 'Save Changes' : 'Add Session'}
    </button>
</form>

<section>
    <h2>Session History</h2>

    {#if sessions.length === 0}
        <p>No sessions logged yet.</p>
    {:else}
        {#each sessions as session}
            <article>
                <h3>{session.game_type} at {session.location}</h3>
                <p>{session.date} · {session.session_type}</p>
                <p>Record: {session.wins}W - {session.losses}L</p>
                <p>{session.notes}</p>

                <button type="button" onclick={() => startEdit(session)}>
                    Edit
                </button>

                <button type="button" class="delete-button" onclick={() => deleteSession(session.id)}>
                    Delete
                </button>
            </article>
        {/each}
    {/if}
</section>
</main>

<style>
    :global(body) {
        margin: 0;
        font-family: system-ui, sans-serif;
        background: #f4f1ea;
        color: #1f2933;
    }

    main {
        max-width: 900px;
        margin: 0 auto;
        padding: 2rem;
    }

    form {
        display: grid;
        gap: 1rem;
        padding: 1.5rem;
        background: white;
        border-radius: 1rem;
        box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);
    }

    label {
        display: grid;
        gap: 0.35rem;
        font-weight: 600;
    }

    input,
    select,
    textarea {
        padding: 0.7rem;
        border: 1px solid #d1d5db;
        border-radius: 0.5rem;
        font: inherit;
    }

    button {
        padding: 0.8rem 1rem;
        border: none;
        border-radius: 0.6rem;
        font-weight: 700;
        cursor: pointer;
    }

    article {
        margin-top: 1rem;
        padding: 1.25rem;
        background: white;
        border-radius: 1rem;
        box-shadow: 0 8px 20px rgba(0, 0, 0, 0.06);
    }

    .delete-button {
        margin-top: 0.75rem;
        background: #fee2e2;
        color: #991b1b;
    }

    .delete-button:hover {
        background: #fecaca;
    }
</style>
