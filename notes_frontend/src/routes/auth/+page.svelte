<script lang="ts">
    import { createEventDispatcher } from 'svelte';
    import { auth } from '$lib/stores/auth';
    import { login, signup } from '$lib/services/api';

    const dispatch = createEventDispatcher();
    let mode: 'login' | 'signup' = 'login';
    let email = '';
    let password = '';
    let loading = false;
    let error: string | null = null;

    async function submit() {
        loading = true;
        error = null;
        try {
            const fn = mode === 'login' ? login : signup;
            const res = await fn(email, password);
            auth.login(res.token, res.user);
            dispatch('done');
            // navigate by setting hash (SvelteKit navigation is automatic via anchor)
            window.location.href = '/';
        } catch (e) {
            error = (e as Error).message;
        } finally {
            loading = false;
        }
    }
</script>

<svelte:head>
    <title>{mode === 'login' ? 'Sign in' : 'Create account'} • Notes</title>
</svelte:head>

<div class="auth-wrap">
    <div class="card auth-card">
        <h1>{mode === 'login' ? 'Welcome back' : 'Create your account'}</h1>
        <p class="text-muted">{mode === 'login' ? 'Sign in to continue' : 'Sign up to start taking notes'}</p>

        {#if error}
            <div class="error">{error}</div>
        {/if}

        <form on:submit|preventDefault={submit}>
            <label for="email">Email</label>
            <input id="email" name="email" type="email" bind:value={email} placeholder="you@example.com" required />

            <label for="password">Password</label>
            <input id="password" name="password" type="password" bind:value={password} placeholder="••••••••" minlength={6} required />

            <button class="btn btn-primary" disabled={loading} type="submit" style="width: 100%; margin-top: 10px;">
                {#if loading}Processing…{/if}
                {#if !loading}{mode === 'login' ? 'Sign in' : 'Sign up'}{/if}
            </button>
        </form>

        <div class="switch text-muted">
            {#if mode === 'login'}
                Don't have an account?
                <button class="btn btn-outline" on:click={() => (mode = 'signup')}>Create one</button>
            {:else}
                Already have an account?
                <button class="btn btn-outline" on:click={() => (mode = 'login')}>Sign in</button>
            {/if}
        </div>
    </div>
</div>

<style>
.auth-wrap {
    display: grid;
    place-items: center;
    min-height: calc(100dvh - 36px);
}
.auth-card {
    width: 100%;
    max-width: 420px;
    padding: 20px;
}
h1 { margin: 0 0 4px 0; font-size: 22px; }
label { display: block; font-size: 13px; color: var(--color-text-secondary); margin: 10px 0 4px; }
input { margin-bottom: 6px; }
.error {
    background: #ffebee;
    color: #b71c1c;
    border: 1px solid #ffcdd2;
    border-radius: var(--radius-sm);
    padding: 10px 12px;
    font-size: 14px;
}
.switch {
    display: flex; justify-content: center; gap: 8px; align-items: center;
    font-size: 14px; margin-top: 10px;
}
</style>
