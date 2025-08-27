<script lang="ts">
    import '../app.css';
    import { auth } from '$lib/stores/auth';
    let { children } = $props();

    function handleLogout() {
        auth.logout();
        // optional: redirect by updating state in pages
    }
</script>

<div class="shell">
    <aside class="sidebar">
        <div class="brand">
            <div class="logo">✦</div>
            <div class="title">
                <strong>Notes</strong>
                <span class="muted">Minimal</span>
            </div>
        </div>

        <nav class="nav">
            <a class="nav-item active" href="/" aria-current="page">
                <span>All Notes</span>
            </a>
        </nav>

        <div class="spacer"></div>

        <div class="auth card">
            {#if $auth.user}
                <div class="user">
                    <div class="avatar">{$auth.user.email.charAt(0).toUpperCase()}</div>
                    <div class="meta">
                        <div class="email">{$auth.user.email}</div>
                        <div class="status">Signed in</div>
                    </div>
                </div>
                <button class="btn btn-outline" onclick={handleLogout}>Sign out</button>
            {:else}
                <div class="text-muted" style="font-size: 14px;">Not signed in</div>
                <a href="/auth" class="btn btn-primary" style="margin-top: 8px;">Sign in</a>
            {/if}
        </div>
    </aside>

    <main class="content">
        {@render children()}
    </main>
</div>

<style>
.shell {
    display: grid;
    grid-template-columns: var(--sidebar-width) 1fr;
    min-height: 100vh;
    background: var(--color-background);
    color: var(--color-text-primary);
}

.sidebar {
    background: linear-gradient(180deg, #ffffff, #f9fafb);
    border-right: 1px solid var(--color-border);
    padding: 18px;
    display: flex;
    flex-direction: column;
    gap: 16px;
}

.brand {
    display: flex;
    align-items: center;
    gap: 10px;
    padding-bottom: 12px;
    border-bottom: 1px solid var(--color-border);
}
.logo {
    width: 36px; height: 36px;
    display: grid; place-items: center;
    background: var(--color-primary);
    color: var(--color-primary-contrast);
    border-radius: 10px;
    box-shadow: var(--shadow-sm);
    font-weight: 700;
}
.title { display: flex; flex-direction: column; line-height: 1.1; }
.title .muted { color: var(--color-text-secondary); font-size: 12px; }

.nav { display: flex; flex-direction: column; gap: 6px; }
.nav-item {
    display: flex;
    align-items: center;
    gap: 10px;
    padding: 10px 12px;
    border-radius: var(--radius-sm);
    color: var(--color-text-primary);
}
.nav-item:hover { background: var(--color-surface); }
.nav-item.active { background: #e8f2fd; color: var(--color-primary); font-weight: 600; }

.spacer { flex: 1; }

.auth {
    padding: 12px;
    display: flex;
    gap: 10px;
    flex-direction: column;
}
.user { display: flex; gap: 10px; align-items: center; }
.avatar {
    width: 34px; height: 34px; border-radius: 10px; display: grid; place-items: center;
    background: var(--color-accent);
    color: #1a1a1a; font-weight: 700;
}
.meta .email { font-size: 14px; }
.meta .status { font-size: 12px; color: var(--color-text-secondary); }

.content {
    min-width: 0;
    padding: 18px;
}

@media (max-width: 980px) {
    .shell { grid-template-columns: 1fr; }
    .sidebar { position: sticky; top: 0; z-index: 5; backdrop-filter: saturate(180%) blur(6px); }
}
</style>
