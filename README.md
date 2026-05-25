<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Usako – Twitch Command Manager</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Syne:wght@400;600;700;800&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet">
<style>
  :root {
    --bg: #0d0f14;
    --surface: #151820;
    --surface2: #1c2030;
    --surface3: #232840;
    --accent: #6c5ce7;
    --accent2: #a29bfe;
    --green: #00b894;
    --red: #e17055;
    --yellow: #fdcb6e;
    --text: #eaeaf5;
    --muted: #7c82a0;
    --border: rgba(108,92,231,0.2);
    --border2: rgba(255,255,255,0.07);
    --radius: 10px;
    --font: 'Syne', sans-serif;
    --mono: 'JetBrains Mono', monospace;
  }
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body { background: var(--bg); color: var(--text); font-family: var(--font); min-height: 100vh; }

  #app { display: flex; flex-direction: column; height: 100vh; overflow: hidden; }
  header { display: flex; align-items: center; gap: 16px; padding: 0 24px; height: 56px; background: var(--surface); border-bottom: 1px solid var(--border2); flex-shrink: 0; }
  header h1 { font-size: 18px; font-weight: 800; letter-spacing: -0.5px; color: var(--accent2); }
  header h1 span { color: var(--text); }
  .main { display: flex; flex: 1; overflow: hidden; }
  .sidebar { width: 220px; background: var(--surface); border-right: 1px solid var(--border2); display: flex; flex-direction: column; flex-shrink: 0; }
  .content { flex: 1; overflow-y: auto; padding: 24px; }
  .panel { display: none; }
  .panel.active { display: block; }

  .nav-section { padding: 16px 12px 8px; font-size: 10px; font-weight: 700; letter-spacing: 1.5px; color: var(--muted); text-transform: uppercase; }
  .nav-item { display: flex; align-items: center; gap: 10px; padding: 9px 16px; font-size: 14px; font-weight: 600; color: var(--muted); cursor: pointer; border-radius: 6px; margin: 1px 6px; transition: all 0.15s; }
  .nav-item:hover { background: var(--surface2); color: var(--text); }
  .nav-item.active { background: rgba(108,92,231,0.15); color: var(--accent2); }
  .nav-item svg { width: 16px; height: 16px; flex-shrink: 0; }

  .accounts-bar { padding: 16px; border-top: 1px solid var(--border2); margin-top: auto; display: flex; flex-direction: column; gap: 8px; }
  .account-chip { display: flex; align-items: center; gap: 8px; padding: 8px 10px; border-radius: 8px; background: var(--surface2); font-size: 12px; }
  .account-chip .dot { width: 8px; height: 8px; border-radius: 50%; background: var(--muted); flex-shrink: 0; }
  .account-chip .dot.online { background: var(--green); }
  .account-chip .name { font-weight: 600; flex: 1; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
  .account-chip .role { font-size: 10px; color: var(--muted); }

  .btn { display: inline-flex; align-items: center; gap: 6px; padding: 8px 16px; border-radius: var(--radius); border: none; font-family: var(--font); font-size: 13px; font-weight: 700; cursor: pointer; transition: all 0.15s; }
  .btn-primary { background: var(--accent); color: #fff; }
  .btn-primary:hover { background: #7c6df0; }
  .btn-ghost { background: transparent; color: var(--muted); border: 1px solid var(--border2); }
  .btn-ghost:hover { color: var(--text); border-color: var(--border); }
  .btn-danger { background: rgba(225,112,85,0.15); color: var(--red); border: 1px solid rgba(225,112,85,0.3); }
  .btn-danger:hover { background: rgba(225,112,85,0.25); }
  .btn-green { background: var(--green); color: #fff; }
  .btn-sm { padding: 5px 10px; font-size: 12px; }
  .btn-icon { padding: 6px; border-radius: 6px; background: transparent; border: 1px solid var(--border2); color: var(--muted); cursor: pointer; transition: all 0.15s; display: flex; align-items: center; justify-content: center; }
  .btn-icon:hover { color: var(--text); border-color: var(--border); }
  .btn-icon svg { width: 14px; height: 14px; }

  input, select, textarea { background: var(--surface2); border: 1px solid var(--border2); color: var(--text); font-family: var(--font); font-size: 13px; border-radius: 8px; padding: 9px 12px; width: 100%; transition: border-color 0.15s; }
  input:focus, select:focus, textarea:focus { outline: none; border-color: var(--accent); }
  label { font-size: 12px; font-weight: 700; color: var(--muted); letter-spacing: 0.5px; display: block; margin-bottom: 6px; }
  .field { margin-bottom: 16px; }
  .toggle { display: flex; align-items: center; gap: 10px; cursor: pointer; }
  .toggle input[type=checkbox] { display: none; }
  .toggle-track { width: 36px; height: 20px; background: var(--surface3); border-radius: 20px; position: relative; transition: background 0.2s; flex-shrink: 0; }
  .toggle-track::after { content: ''; position: absolute; left: 2px; top: 2px; width: 16px; height: 16px; background: var(--muted); border-radius: 50%; transition: all 0.2s; }
  .toggle input:checked + .toggle-track { background: var(--accent); }
  .toggle input:checked + .toggle-track::after { left: 18px; background: #fff; }
  .toggle-label { font-size: 13px; font-weight: 600; color: var(--text); }

  .card { background: var(--surface); border: 1px solid var(--border2); border-radius: 12px; padding: 20px; margin-bottom: 20px; }
  .card-header { display: flex; align-items: center; justify-content: space-between; margin-bottom: 16px; }
  .card-title { font-size: 15px; font-weight: 700; }
  .card-subtitle { font-size: 12px; color: var(--muted); margin-top: 2px; }

  .badge { display: inline-flex; align-items: center; gap: 4px; padding: 2px 8px; border-radius: 20px; font-size: 10px; font-weight: 700; letter-spacing: 0.5px; text-transform: uppercase; }
  .badge-purple { background: rgba(108,92,231,0.2); color: var(--accent2); }
  .badge-green { background: rgba(0,184,148,0.15); color: var(--green); }
  .badge-red { background: rgba(225,112,85,0.15); color: var(--red); }
  .badge-yellow { background: rgba(253,203,110,0.15); color: var(--yellow); }

  .cmd-list { display: flex; flex-direction: column; gap: 8px; }
  .cmd-row { background: var(--surface); border: 1px solid var(--border2); border-radius: 10px; padding: 14px 16px; display: flex; align-items: center; gap: 12px; cursor: pointer; transition: all 0.15s; }
  .cmd-row:hover { border-color: var(--border); background: var(--surface2); }
  .cmd-row.selected { border-color: var(--accent); background: rgba(108,92,231,0.08); }
  .cmd-name { font-weight: 700; font-family: var(--mono); font-size: 14px; color: var(--accent2); flex: 1; }
  .cmd-triggers { font-size: 11px; color: var(--muted); margin-top: 2px; }
  .cmd-actions-count { font-size: 11px; color: var(--muted); flex-shrink: 0; }

  .action-block { background: var(--surface2); border: 1px solid var(--border2); border-radius: 10px; overflow: hidden; margin-bottom: 10px; }
  .action-header { display: flex; align-items: center; gap: 10px; padding: 10px 14px; cursor: pointer; user-select: none; }
  .action-header:hover { background: var(--surface3); }
  .action-type-dot { width: 10px; height: 10px; border-radius: 50%; flex-shrink: 0; }
  .dot-chat { background: var(--accent2); }
  .dot-sound { background: var(--green); }
  .dot-overlay { background: var(--yellow); }
  .action-type-name { font-size: 13px; font-weight: 700; flex: 1; }
  .action-body { padding: 12px 14px; border-top: 1px solid var(--border2); display: none; }
  .action-block.open .action-body { display: block; }
  .action-expand { width: 16px; height: 16px; color: var(--muted); transition: transform 0.2s; }
  .action-block.open .action-expand { transform: rotate(90deg); }

  #chat-log { background: var(--surface2); border: 1px solid var(--border2); border-radius: 10px; padding: 12px; height: 300px; overflow-y: auto; font-family: var(--mono); font-size: 12px; display: flex; flex-direction: column; gap: 4px; }
  .chat-msg { display: flex; gap: 8px; padding: 3px 0; }
  .chat-user { font-weight: 700; color: var(--accent2); flex-shrink: 0; }
  .chat-text { color: var(--text); word-break: break-word; }
  .chat-cmd { color: var(--green); }
  .chat-system { color: var(--muted); font-style: italic; }
  .conn-badge { display: flex; align-items: center; gap: 6px; font-size: 12px; font-weight: 600; }
  .conn-dot { width: 8px; height: 8px; border-radius: 50%; background: var(--red); }
  .conn-dot.connected { background: var(--green); animation: pulse 2s infinite; }
  @keyframes pulse { 0%,100%{opacity:1}50%{opacity:0.5} }

  #overlay-container { position: fixed; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; z-index: 9999; }
  #overlay-container img, #overlay-container video { position: absolute; max-width: 100%; max-height: 100%; }

  .modal-bg { position: fixed; inset: 0; background: rgba(0,0,0,0.7); z-index: 1000; display: flex; align-items: center; justify-content: center; padding: 20px; }
  .modal { background: var(--surface); border: 1px solid var(--border); border-radius: 14px; width: 100%; max-width: 640px; max-height: 90vh; overflow-y: auto; }
  .modal-header { padding: 20px 24px 0; display: flex; align-items: center; justify-content: space-between; }
  .modal-body { padding: 20px 24px; }
  .modal-footer { padding: 0 24px 20px; display: flex; justify-content: flex-end; gap: 10px; }
  .modal-title { font-size: 18px; font-weight: 800; }

  #login-screen { position: fixed; inset: 0; background: var(--bg); z-index: 500; display: flex; align-items: center; justify-content: center; }
  .login-box { background: var(--surface); border: 1px solid var(--border); border-radius: 16px; padding: 40px; width: 460px; text-align: center; }
  .login-box h2 { font-size: 28px; font-weight: 800; margin-bottom: 8px; }
  .login-box p { color: var(--muted); font-size: 14px; margin-bottom: 32px; }
  .twitch-btn { display: flex; align-items: center; justify-content: center; gap: 10px; width: 100%; padding: 14px; border-radius: 10px; background: #9146FF; color: white; font-family: var(--font); font-size: 15px; font-weight: 700; border: none; cursor: pointer; transition: background 0.15s; }
  .twitch-btn:hover { background: #7d2df5; }
  .twitch-btn svg { width: 20px; height: 20px; }
  .divider { display: flex; align-items: center; gap: 12px; margin: 20px 0; color: var(--muted); font-size: 12px; }
  .divider::before, .divider::after { content: ''; flex: 1; height: 1px; background: var(--border2); }
  .skip-link { color: var(--muted); font-size: 13px; cursor: pointer; text-decoration: underline; }
  .skip-link:hover { color: var(--text); }

  .file-chip { display: flex; align-items: center; gap: 8px; padding: 6px 10px; background: var(--surface3); border-radius: 6px; font-size: 12px; font-family: var(--mono); margin-top: 8px; }
  .file-chip .del { color: var(--red); cursor: pointer; margin-left: auto; }

  .row { display: flex; gap: 12px; }
  .row > * { flex: 1; }
  .section-title { font-size: 22px; font-weight: 800; margin-bottom: 4px; }
  .section-sub { font-size: 14px; color: var(--muted); margin-bottom: 24px; }
  .empty-state { text-align: center; padding: 60px 20px; color: var(--muted); }
  .empty-state svg { width: 48px; height: 48px; margin-bottom: 16px; opacity: 0.3; }
  .empty-state p { font-size: 14px; }
  hr { border: none; border-top: 1px solid var(--border2); margin: 20px 0; }
  .tag { display: inline-block; padding: 2px 8px; border-radius: 4px; background: var(--surface3); font-size: 11px; font-family: var(--mono); color: var(--muted); margin: 2px; }
  select option { background: var(--surface2); }
  .alert { padding: 12px 16px; border-radius: 8px; font-size: 13px; margin-bottom: 16px; }
  .alert-info { background: rgba(108,92,231,0.1); border: 1px solid rgba(108,92,231,0.3); color: var(--accent2); }
  .alert-warning { background: rgba(253,203,110,0.1); border: 1px solid rgba(253,203,110,0.3); color: var(--yellow); }
  ::-webkit-scrollbar { width: 6px; } ::-webkit-scrollbar-track { background: transparent; } ::-webkit-scrollbar-thumb { background: var(--surface3); border-radius: 3px; }
</style>
</head>
<body>

<div id="overlay-container"></div>

<!-- LOGIN SCREEN -->
<div id="login-screen">
  <div class="login-box">
    <div style="font-size:36px;margin-bottom:12px">🐰</div>
    <h2>Usako</h2>
    <p>Connect your Twitch accounts to start managing chat commands</p>
    <button class="twitch-btn" id="btn-login-streamer">
      <svg viewBox="0 0 24 24" fill="currentColor"><path d="M11.571 4.714h1.715v5.143H11.57zm4.715 0H18v5.143h-1.714zM6 0L1.714 4.286v15.428h5.143V24l4.286-4.286h3.428L22.286 12V0zm14.571 11.143l-3.428 3.428h-3.429l-3 3v-3H6.857V1.714h13.714z"/></svg>
      Connect Streamer Account
    </button>
    <div class="divider">Bot Account (Optional)</div>
    <button class="twitch-btn" id="btn-login-bot" style="background:#6c5ce7">
      <svg viewBox="0 0 24 24" fill="currentColor"><path d="M11.571 4.714h1.715v5.143H11.57zm4.715 0H18v5.143h-1.714zM6 0L1.714 4.286v15.428h5.143V24l4.286-4.286h3.428L22.286 12V0zm14.571 11.143l-3.428 3.428h-3.429l-3 3v-3H6.857V1.714h13.714z"/></svg>
      Connect Bot Account
    </button>
    <div style="margin-top:16px">
      <span class="skip-link" id="skip-login">Skip bot account / continue without login (demo mode)</span>
    </div>
    <div style="margin-top:12px">
      <button class="btn btn-danger btn-sm" id="btn-clear-data-login" style="width:100%;justify-content:center">🗑 Clear Saved Data</button>
    </div>
  </div>
</div>

<!-- MAIN APP -->
<div id="app" style="display:none">
  <header>
    <h1>Usa<span>ko</span></h1>
    <div style="margin-left:auto;display:flex;align-items:center;gap:16px">
      <div class="conn-badge">
        <div class="conn-dot" id="conn-dot"></div>
        <span id="conn-label" style="color:var(--muted)">Disconnected</span>
      </div>
      <button class="btn btn-primary btn-sm" id="btn-connect-chat">Connect to Chat</button>
    </div>
  </header>
  <div class="main">
    <div class="sidebar">
      <div class="nav-section">Menu</div>
      <div class="nav-item active" data-panel="commands">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="4 17 10 11 4 5"/><line x1="12" y1="19" x2="20" y2="19"/></svg>
        Commands
      </div>
      <div class="nav-item" data-panel="chat">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"/></svg>
        Chat Monitor
      </div>
      <div class="nav-item" data-panel="media">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="3" y="3" width="18" height="18" rx="2"/><circle cx="8.5" cy="8.5" r="1.5"/><polyline points="21 15 16 10 5 21"/></svg>
        Media Library
      </div>
      <div class="nav-item" data-panel="settings">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="3"/><path d="M19.07 4.93a10 10 0 0 0-14.14 0M4.93 19.07a10 10 0 0 0 14.14 0M12 2v2m0 18v2M2 12h2m18 0h2"/></svg>
        Settings
      </div>
      <div class="accounts-bar">
        <div class="account-chip">
          <div class="dot" id="dot-streamer"></div>
          <div><div class="name" id="name-streamer">Not connected</div><div class="role">Streamer</div></div>
        </div>
        <div class="account-chip">
          <div class="dot" id="dot-bot"></div>
          <div><div class="name" id="name-bot">No bot</div><div class="role">Bot</div></div>
        </div>
      </div>
    </div>

    <div class="content">

      <!-- COMMANDS PANEL -->
      <div class="panel active" id="panel-commands">
        <div style="display:flex;align-items:flex-start;justify-content:space-between;margin-bottom:24px">
          <div>
            <div class="section-title">Commands</div>
            <div class="section-sub">Manage your chat commands and their actions</div>
          </div>
          <button class="btn btn-primary" id="btn-new-cmd">
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><line x1="12" y1="5" x2="12" y2="19"/><line x1="5" y1="12" x2="19" y2="12"/></svg>
            New Command
          </button>
        </div>
        <div id="cmd-list-container">
          <div class="empty-state" id="cmd-empty">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><polyline points="4 17 10 11 4 5"/><line x1="12" y1="19" x2="20" y2="19"/></svg>
            <p>No commands yet.<br>Click <strong>New Command</strong> to create your first one.</p>
          </div>
          <div class="cmd-list" id="cmd-list"></div>
        </div>
      </div>

      <!-- CHAT PANEL -->
      <div class="panel" id="panel-chat">
        <div class="section-title">Chat Monitor</div>
        <div class="section-sub">Live Twitch chat — commands will be highlighted</div>
        <div class="card" style="margin-bottom:0">
          <div class="card-header">
            <span>Live Chat</span>
            <button class="btn btn-ghost btn-sm" id="btn-clear-chat">Clear</button>
          </div>
          <div id="chat-log"><div class="chat-msg"><span class="chat-system">Waiting for chat connection…</span></div></div>
        </div>
      </div>

      <!-- MEDIA PANEL -->
      <div class="panel" id="panel-media">
        <div class="section-title">Media Library</div>
        <div class="section-sub">Uploaded sounds and overlay media — used by your commands</div>
        <div class="row">
          <div class="card">
            <div class="card-header"><span class="card-title">🔊 Sounds</span><button class="btn btn-ghost btn-sm" id="btn-upload-sound">Upload</button></div>
            <input type="file" id="inp-upload-sound" accept="audio/*" style="display:none">
            <div id="sound-library"></div>
          </div>
          <div class="card">
            <div class="card-header"><span class="card-title">🖼️ Overlays</span><button class="btn btn-ghost btn-sm" id="btn-upload-overlay">Upload</button></div>
            <input type="file" id="inp-upload-overlay" accept="image/*,video/*" style="display:none">
            <div id="overlay-library"></div>
          </div>
        </div>
      </div>

      <!-- SETTINGS PANEL -->
      <div class="panel" id="panel-settings">
        <div class="section-title">Settings</div>
        <div class="section-sub">Account management</div>
        <div class="card">
          <div class="card-title" style="margin-bottom:16px">Accounts</div>
          <div class="field">
            <label>Streamer Account</label>
            <div style="display:flex;gap:10px;align-items:center">
              <input type="text" id="set-streamer-name" readonly placeholder="Not connected" style="flex:1">
              <button class="btn btn-ghost btn-sm" id="btn-set-streamer">Change</button>
              <button class="btn btn-danger btn-sm" id="btn-logout-streamer">Logout</button>
            </div>
          </div>
          <div class="field">
            <label>Bot Account</label>
            <div style="display:flex;gap:10px;align-items:center">
              <input type="text" id="set-bot-name" readonly placeholder="No bot account" style="flex:1">
              <button class="btn btn-ghost btn-sm" id="btn-set-bot">Change</button>
              <button class="btn btn-danger btn-sm" id="btn-logout-bot">Remove</button>
            </div>
          </div>
          <div class="alert alert-info">
            Tokens are stored only in your browser's localStorage and never sent to any server.
          </div>
        </div>
        <div class="card">
          <div class="card-title" style="margin-bottom:16px">Danger Zone</div>
          <button class="btn btn-danger" id="btn-clear-all">Clear All Data</button>
        </div>
      </div>

    </div>
  </div>
</div>

<!-- COMMAND EDITOR MODAL -->
<div class="modal-bg" id="cmd-modal" style="display:none">
  <div class="modal">
    <div class="modal-header">
      <div class="modal-title" id="modal-title">New Command</div>
      <button class="btn-icon" id="btn-modal-close"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg></button>
    </div>
    <div class="modal-body">
      <div class="row">
        <div class="field">
          <label>Command Name</label>
          <input type="text" id="cmd-name" placeholder="e.g. beanuncle">
        </div>
        <div class="field">
          <label>Command Group</label>
          <input type="text" id="cmd-group" placeholder="e.g. Greetings">
        </div>
      </div>
      <div class="field">
        <label>Chat Triggers (space separated; semicolon for multi-word)</label>
        <input type="text" id="cmd-triggers" placeholder="e.g. beanunc beanuncle uncbean">
      </div>
      <div class="row" style="align-items:center;margin-bottom:16px">
        <label class="toggle"><input type="checkbox" id="cmd-auto-exclaim" checked><div class="toggle-track"></div><span class="toggle-label">Auto-Include "!"</span></label>
        <label class="toggle"><input type="checkbox" id="cmd-wildcards"><div class="toggle-track"></div><span class="toggle-label">Wildcards</span></label>
        <label class="toggle"><input type="checkbox" id="cmd-enabled" checked><div class="toggle-track"></div><span class="toggle-label">Enabled</span></label>
      </div>
      <hr>
      <div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:12px">
        <div style="font-size:13px;font-weight:700">Actions</div>
        <div style="display:flex;gap:8px">
          <button class="btn btn-ghost btn-sm" data-add-action="chat">+ Chat Message</button>
          <button class="btn btn-ghost btn-sm" data-add-action="sound">+ Sound</button>
          <button class="btn btn-ghost btn-sm" data-add-action="overlay">+ Overlay</button>
        </div>
      </div>
      <div id="action-list"></div>
    </div>
    <div class="modal-footer">
      <button class="btn btn-ghost" id="btn-modal-cancel">Cancel</button>
      <button class="btn btn-primary" id="btn-modal-save">Save Command</button>
    </div>
  </div>
</div>

<script>
// ─── CONFIG ───────────────────────────────────────────────────────────────────
// IMPORTANT: Replace with your own Twitch App Client ID.
// Register your app at https://dev.twitch.tv/console
// Set the OAuth Redirect URL to: https://usagitsukiii.neocities.org/bot
const CLIENT_ID = 'tybrj9y1ts5eas6cyrl1j0diw2v7ln';
const REDIRECT_URI = 'https://usagi-tsukiii.github.io/usako.html';

// All requested Twitch scopes
const SCOPES = [
  'channel:edit:commercial',
  'channel:manage:broadcast',
  'channel:manage:moderators',
  'channel:manage:redemptions',
  'channel:read:vips',
  'clips:edit',
  'channel:read:editors',
  'channel:read:redemptions',
  'channel:read:subscriptions',
  'chat:read',
  'chat:edit',
  'moderation:read',
  'moderator:manage:banned_users',
  'moderator:read:followers',
  'user:read:email'
].join(' ');

// ─── STATE ───────────────────────────────────────────────────────────────────
const STORE_KEY = 'usako_v1';
let state = {
  streamer: null,
  bot: null,
  commands: [],
  mediaLib: { sounds: [], overlays: [] }
};

function loadState() {
  try {
    const s = localStorage.getItem(STORE_KEY);
    if (s) state = { ...state, ...JSON.parse(s) };
  } catch(e) {}
}
function saveState() {
  try { localStorage.setItem(STORE_KEY, JSON.stringify(state)); } catch(e) { console.warn('Storage full?', e); }
}

// ─── OAUTH ───────────────────────────────────────────────────────────────────
function doOAuth(role) {
  localStorage.setItem('usako_pending_role', role);
  const scopes = encodeURIComponent(SCOPES);
  const redirect = encodeURIComponent(REDIRECT_URI);
  const url = 'https://id.twitch.tv/oauth2/authorize?client_id=' + CLIENT_ID + '&redirect_uri=' + redirect + '&response_type=token&scope=' + scopes + '&force_verify=true';
  window.location.href = url;
}

function handleOAuthCallback() {
  const hash = location.hash;
  if (!hash.includes('access_token')) return false;
  const params = new URLSearchParams(hash.substring(1));
  const token = params.get('access_token');
  const role = localStorage.getItem('usako_pending_role');
  history.replaceState(null, '', location.pathname);
  if (!token) { alert('Login failed: no access token returned. Please try again.'); showLogin(); return true; }
  const effectiveRole = role || 'streamer';
  localStorage.removeItem('usako_pending_role');
  // No fetch needed — prompt for channel name and store the token directly.
  // The token is valid (Twitch only redirects with one if auth succeeded).
  const defaultName = effectiveRole === 'streamer' ? 'your_channel' : 'your_bot';
  const name = prompt((effectiveRole === 'streamer' ? 'Streamer' : 'Bot') + ' account connected! Enter the Twitch username for this account:', defaultName);
  if (!name || !name.trim()) { showLogin(); return true; }
  const account = { name: name.trim().toLowerCase(), display: name.trim(), token };
  if (effectiveRole === 'streamer') state.streamer = account;
  else state.bot = account;
  saveState();
  showApp();
  return true;
}

function enterDemoMode() {
  const name = prompt('Enter your Twitch channel name for demo mode:') || 'mychannel';
  state.streamer = { name: name.toLowerCase(), display: name, token: null, demo: true };
  saveState();
  showApp();
}

// ─── UI ───────────────────────────────────────────────────────────────────────
function showLogin() {
  document.getElementById('login-screen').style.display = 'flex';
  document.getElementById('app').style.display = 'none';
}

function showApp() {
  document.getElementById('login-screen').style.display = 'none';
  document.getElementById('app').style.display = 'flex';
  updateAccountChips();
  renderCommands();
  renderMediaLib();
}

function updateAccountChips() {
  const sName = document.getElementById('name-streamer');
  const sDot = document.getElementById('dot-streamer');
  const bName = document.getElementById('name-bot');
  const bDot = document.getElementById('dot-bot');
  if (state.streamer) { sName.textContent = state.streamer.display || state.streamer.name; sDot.classList.add('online'); }
  else { sName.textContent = 'Not connected'; sDot.classList.remove('online'); }
  if (state.bot) { bName.textContent = state.bot.display || state.bot.name; bDot.classList.add('online'); }
  else { bName.textContent = 'No bot'; bDot.classList.remove('online'); }
  document.getElementById('set-streamer-name').value = state.streamer ? (state.streamer.display || state.streamer.name) : '';
  document.getElementById('set-bot-name').value = state.bot ? (state.bot.display || state.bot.name) : '';
}

// ─── NAVIGATION ───────────────────────────────────────────────────────────────
document.querySelectorAll('.nav-item').forEach(el => {
  el.addEventListener('click', () => {
    document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active'));
    document.querySelectorAll('.panel').forEach(p => p.classList.remove('active'));
    el.classList.add('active');
    document.getElementById('panel-' + el.dataset.panel).classList.add('active');
  });
});

// ─── COMMANDS ─────────────────────────────────────────────────────────────────
function renderCommands() {
  const list = document.getElementById('cmd-list');
  const empty = document.getElementById('cmd-empty');
  list.innerHTML = '';
  if (!state.commands.length) { empty.style.display = 'block'; return; }
  empty.style.display = 'none';
  state.commands.forEach((cmd, i) => {
    const row = document.createElement('div');
    row.className = 'cmd-row';
    const triggers = buildTriggers(cmd);
    const actionCount = (cmd.actions || []).length;
    row.innerHTML = `
      <div style="flex:1">
        <div class="cmd-name">${escHtml(cmd.name)}</div>
        <div class="cmd-triggers">${triggers.map(t => `<span class="tag">${escHtml(t)}</span>`).join('')}</div>
      </div>
      <div class="cmd-actions-count">${actionCount} action${actionCount !== 1 ? 's' : ''}</div>
      <span class="badge ${cmd.enabled !== false ? 'badge-green' : 'badge-red'}">${cmd.enabled !== false ? 'on' : 'off'}</span>
      <button class="btn-icon edit-btn" data-i="${i}" title="Edit"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"/><path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"/></svg></button>
      <button class="btn-icon del-btn" data-i="${i}" title="Delete" style="color:var(--red)"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="3 6 5 6 21 6"/><path d="M19 6l-1 14a2 2 0 0 1-2 2H8a2 2 0 0 1-2-2L5 6"/><path d="M10 11v6M14 11v6"/><path d="M9 6V4a1 1 0 0 1 1-1h4a1 1 0 0 1 1 1v2"/></svg></button>
    `;
    list.appendChild(row);
  });
  list.querySelectorAll('.edit-btn').forEach(b => b.addEventListener('click', e => { e.stopPropagation(); openCmdModal(parseInt(b.dataset.i)); }));
  list.querySelectorAll('.del-btn').forEach(b => b.addEventListener('click', e => { e.stopPropagation(); deleteCmd(parseInt(b.dataset.i)); }));
}

function buildTriggers(cmd) {
  const raw = (cmd.triggers || '').trim().split(/\s+/).filter(Boolean);
  if (!raw.length) raw.push(cmd.name);
  if (cmd.autoExclaim !== false) return raw.map(t => '!' + t);
  return raw;
}

function deleteCmd(i) {
  if (!confirm('Delete this command?')) return;
  state.commands.splice(i, 1);
  saveState();
  renderCommands();
}

// ─── COMMAND MODAL ────────────────────────────────────────────────────────────
let editingIndex = -1;

function openCmdModal(index) {
  editingIndex = index;
  const isNew = index === -1;
  document.getElementById('modal-title').textContent = isNew ? 'New Command' : 'Edit Command';
  const cmd = isNew ? { name: '', group: '', triggers: '', autoExclaim: true, wildcards: false, enabled: true, actions: [] } : state.commands[index];
  document.getElementById('cmd-name').value = cmd.name || '';
  document.getElementById('cmd-group').value = cmd.group || '';
  document.getElementById('cmd-triggers').value = cmd.triggers || '';
  document.getElementById('cmd-auto-exclaim').checked = cmd.autoExclaim !== false;
  document.getElementById('cmd-wildcards').checked = !!cmd.wildcards;
  document.getElementById('cmd-enabled').checked = cmd.enabled !== false;
  renderActionList(cmd.actions || []);
  document.getElementById('cmd-modal').style.display = 'flex';
}

function renderActionList(actions) {
  const container = document.getElementById('action-list');
  container.innerHTML = '';
  if (!actions.length) {
    container.innerHTML = '<div style="text-align:center;color:var(--muted);font-size:13px;padding:20px">No actions yet. Add one above.</div>';
    return;
  }
  actions.forEach((action, i) => {
    const block = document.createElement('div');
    block.className = 'action-block';
    block.dataset.i = i;
    const dotClass = action.type === 'chat' ? 'dot-chat' : action.type === 'sound' ? 'dot-sound' : 'dot-overlay';
    const typeName = action.type === 'chat' ? 'Chat Message' : action.type === 'sound' ? 'Sound' : 'Overlay (Images & Videos)';
    block.innerHTML = `
      <div class="action-header">
        <div class="action-type-dot ${dotClass}"></div>
        <div class="action-type-name">${typeName}</div>
        <div style="display:flex;gap:6px;margin-left:auto">
          ${i > 0 ? `<button class="btn-icon move-up-btn" title="Move up"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="18 15 12 9 6 15"/></svg></button>` : ''}
          ${i < actions.length - 1 ? `<button class="btn-icon move-down-btn" title="Move down"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="6 9 12 15 18 9"/></svg></button>` : ''}
          <button class="btn-icon del-action-btn" style="color:var(--red)" title="Remove action"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg></button>
        </div>
        <svg class="action-expand" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="9 18 15 12 9 6"/></svg>
      </div>
      <div class="action-body">${buildActionBody(action, i)}</div>
    `;
    container.appendChild(block);
    block.querySelector('.action-header').addEventListener('click', (e) => {
      if (e.target.closest('button')) return;
      block.classList.toggle('open');
    });
    block.querySelector('.del-action-btn').addEventListener('click', () => { actions.splice(i, 1); renderActionList(actions); });
    const upBtn = block.querySelector('.move-up-btn');
    const dnBtn = block.querySelector('.move-down-btn');
    if (upBtn) upBtn.addEventListener('click', () => { [actions[i-1], actions[i]] = [actions[i], actions[i-1]]; renderActionList(actions); });
    if (dnBtn) dnBtn.addEventListener('click', () => { [actions[i], actions[i+1]] = [actions[i+1], actions[i]]; renderActionList(actions); });
    block.classList.add('open');
  });
  container.querySelectorAll('select[data-action-field]').forEach(sel => {
    sel.addEventListener('change', () => {
      const ai = parseInt(sel.closest('.action-block').dataset.i);
      actions[ai][sel.dataset.actionField] = sel.value;
    });
  });
  container.querySelectorAll('input[data-action-field],textarea[data-action-field]').forEach(inp => {
    inp.addEventListener('input', () => {
      const ai = parseInt(inp.closest('.action-block').dataset.i);
      const field = inp.dataset.actionField;
      const val = inp.type === 'number' ? parseFloat(inp.value) : inp.value;
      actions[ai][field] = val;
    });
  });
}

function buildActionBody(action, i) {
  if (action.type === 'chat') {
    return `
      <div class="field"><label>Message</label><textarea data-action-field="message" rows="3" placeholder="Hello {username}! Use {username} for the triggering user.">${escHtml(action.message || '')}</textarea></div>
      <div style="font-size:11px;color:var(--muted)">Variables: {username} {channel} {args}</div>
    `;
  }
  if (action.type === 'sound') {
    const opts = state.mediaLib.sounds.map(s => `<option value="${s.id}" ${action.fileId === s.id ? 'selected' : ''}>${escHtml(s.name)}</option>`).join('');
    return `
      <div class="field"><label>Sound File</label>
        <select data-action-field="fileId"><option value="">-- Select uploaded sound --</option>${opts}</select>
      </div>
      <div class="row">
        <div class="field"><label>Volume (0–1)</label><input type="number" data-action-field="volume" min="0" max="1" step="0.05" value="${action.volume !== undefined ? action.volume : 1}"></div>
        <div class="field"><label>Delay (ms)</label><input type="number" data-action-field="delay" min="0" step="100" value="${action.delay || 0}"></div>
      </div>
    `;
  }
  if (action.type === 'overlay') {
    const opts = state.mediaLib.overlays.map(o => `<option value="${o.id}" ${action.fileId === o.id ? 'selected' : ''}>${escHtml(o.name)}</option>`).join('');
    return `
      <div class="field"><label>Overlay File</label>
        <select data-action-field="fileId"><option value="">-- Select uploaded image/video --</option>${opts}</select>
      </div>
      <div class="row">
        <div class="field"><label>Position</label>
          <select data-action-field="position">
            ${['center','top-left','top-right','bottom-left','bottom-right','top-center','bottom-center'].map(p => `<option value="${p}" ${action.position === p ? 'selected' : ''}>${p}</option>`).join('')}
          </select>
        </div>
        <div class="field"><label>Duration (ms, 0=manual)</label><input type="number" data-action-field="duration" min="0" step="500" value="${action.duration !== undefined ? action.duration : 3000}"></div>
      </div>
      <div class="row">
        <div class="field"><label>Width (px, 0=auto)</label><input type="number" data-action-field="width" min="0" step="10" value="${action.width || 0}"></div>
        <div class="field"><label>Delay (ms)</label><input type="number" data-action-field="delay" min="0" step="100" value="${action.delay || 0}"></div>
      </div>
    `;
  }
  return '';
}

document.querySelectorAll('[data-add-action]').forEach(btn => {
  btn.addEventListener('click', () => {
    const type = btn.dataset.addAction;
    const currentActions = collectCurrentActions();
    currentActions.push({ type, ...(type === 'sound' ? {volume: 1, delay: 0} : {}), ...(type === 'overlay' ? {duration: 3000, delay: 0, position: 'center', width: 0} : {}) });
    renderActionList(currentActions);
  });
});

function collectCurrentActions() {
  const blocks = document.querySelectorAll('#action-list .action-block');
  const actions = [];
  blocks.forEach(block => {
    const fields = {};
    block.querySelectorAll('[data-action-field]').forEach(el => {
      const val = el.tagName === 'SELECT' || el.type !== 'number' ? el.value : parseFloat(el.value);
      fields[el.dataset.actionField] = val;
    });
    const dot = block.querySelector('.action-type-dot');
    let type = 'chat';
    if (dot.classList.contains('dot-sound')) type = 'sound';
    if (dot.classList.contains('dot-overlay')) type = 'overlay';
    actions.push({ type, ...fields });
  });
  return actions;
}

document.getElementById('btn-new-cmd').addEventListener('click', () => openCmdModal(-1));
document.getElementById('btn-modal-close').addEventListener('click', closeCmdModal);
document.getElementById('btn-modal-cancel').addEventListener('click', closeCmdModal);

function closeCmdModal() { document.getElementById('cmd-modal').style.display = 'none'; }

document.getElementById('btn-modal-save').addEventListener('click', () => {
  const name = document.getElementById('cmd-name').value.trim();
  if (!name) { alert('Command name is required.'); return; }
  const actions = collectCurrentActions();
  const cmd = {
    id: editingIndex === -1 ? Date.now() : state.commands[editingIndex].id,
    name,
    group: document.getElementById('cmd-group').value.trim(),
    triggers: document.getElementById('cmd-triggers').value.trim(),
    autoExclaim: document.getElementById('cmd-auto-exclaim').checked,
    wildcards: document.getElementById('cmd-wildcards').checked,
    enabled: document.getElementById('cmd-enabled').checked,
    actions
  };
  if (editingIndex === -1) state.commands.push(cmd);
  else state.commands[editingIndex] = cmd;
  saveState();
  renderCommands();
  closeCmdModal();
});

// ─── MEDIA LIBRARY ────────────────────────────────────────────────────────────
function renderMediaLib() {
  const sl = document.getElementById('sound-library');
  const ol = document.getElementById('overlay-library');
  sl.innerHTML = '';
  ol.innerHTML = '';
  if (!state.mediaLib.sounds.length) sl.innerHTML = '<div style="color:var(--muted);font-size:12px;text-align:center;padding:16px">No sounds uploaded yet</div>';
  state.mediaLib.sounds.forEach(s => {
    sl.insertAdjacentHTML('beforeend', `<div class="file-chip"><span>🔊</span><span style="flex:1">${escHtml(s.name)}</span><span class="del" data-del-sound="${s.id}">✕</span></div>`);
  });
  if (!state.mediaLib.overlays.length) ol.innerHTML = '<div style="color:var(--muted);font-size:12px;text-align:center;padding:16px">No overlays uploaded yet</div>';
  state.mediaLib.overlays.forEach(o => {
    ol.insertAdjacentHTML('beforeend', `<div class="file-chip"><span>${o.mime.startsWith('video') ? '🎬' : '🖼️'}</span><span style="flex:1">${escHtml(o.name)}</span><span class="del" data-del-overlay="${o.id}">✕</span></div>`);
  });
  sl.querySelectorAll('[data-del-sound]').forEach(el => el.addEventListener('click', () => { state.mediaLib.sounds = state.mediaLib.sounds.filter(s => s.id != el.dataset.delSound); saveState(); renderMediaLib(); }));
  ol.querySelectorAll('[data-del-overlay]').forEach(el => el.addEventListener('click', () => { state.mediaLib.overlays = state.mediaLib.overlays.filter(o => o.id != el.dataset.delOverlay); saveState(); renderMediaLib(); }));
}

function handleFileUpload(file, category) {
  if (!file) return;
  const maxMB = 8;
  if (file.size > maxMB * 1024 * 1024) { alert(`File too large. Max ${maxMB}MB.`); return; }
  const reader = new FileReader();
  reader.onload = e => {
    const item = { id: Date.now().toString(), name: file.name, mime: file.type, b64: e.target.result };
    if (category === 'sound') state.mediaLib.sounds.push(item);
    else state.mediaLib.overlays.push(item);
    saveState();
    renderMediaLib();
  };
  reader.readAsDataURL(file);
}

document.getElementById('btn-upload-sound').addEventListener('click', () => document.getElementById('inp-upload-sound').click());
document.getElementById('btn-upload-overlay').addEventListener('click', () => document.getElementById('inp-upload-overlay').click());
document.getElementById('inp-upload-sound').addEventListener('change', e => handleFileUpload(e.target.files[0], 'sound'));
document.getElementById('inp-upload-overlay').addEventListener('change', e => handleFileUpload(e.target.files[0], 'overlay'));

// ─── SETTINGS ─────────────────────────────────────────────────────────────────
document.getElementById('btn-logout-streamer').addEventListener('click', () => { if (confirm('Logout streamer?')) { state.streamer = null; saveState(); showLogin(); } });
document.getElementById('btn-logout-bot').addEventListener('click', () => { state.bot = null; saveState(); updateAccountChips(); });
document.getElementById('btn-set-streamer').addEventListener('click', () => doOAuth('streamer'));
document.getElementById('btn-set-bot').addEventListener('click', () => doOAuth('bot'));
document.getElementById('btn-clear-all').addEventListener('click', () => { if (confirm('Clear ALL data? This cannot be undone.')) { localStorage.removeItem(STORE_KEY); location.reload(); } });

// Login screen
document.getElementById('btn-login-streamer').addEventListener('click', () => doOAuth('streamer'));
document.getElementById('btn-login-bot').addEventListener('click', () => doOAuth('bot'));
document.getElementById('skip-login').addEventListener('click', () => enterDemoMode());
document.getElementById('btn-clear-data-login').addEventListener('click', () => {
  if (confirm('Clear all saved data? This will remove your accounts, commands, and media library. This cannot be undone.')) {
    localStorage.removeItem(STORE_KEY);
    location.reload();
  }
});

// ─── TWITCH IRC ────────────────────────────────────────────────────────────────
let ircSocket = null;
let isConnected = false;
let reconnectTimeout = null;

function connectIRC() {
  if (ircSocket) { ircSocket.close(); ircSocket = null; }
  if (!state.streamer) { addChatMsg('system', '', 'Login required to connect to chat.'); return; }
  const channel = state.streamer.name;
  const sender = state.bot || state.streamer;
  const token = sender.token;
  const nick = sender.name;

  addChatMsg('system', '', `Connecting to #${channel}…`);
  setConnState(false);

  ircSocket = new WebSocket('wss://irc-ws.chat.twitch.tv:443');
  ircSocket.onopen = () => {
    if (token) {
      ircSocket.send(`PASS oauth:${token}`);
      ircSocket.send(`NICK ${nick}`);
    } else {
      ircSocket.send('PASS SCHMOOPIIE');
      ircSocket.send(`NICK justinfan${Math.floor(Math.random()*99999)}`);
    }
    ircSocket.send('CAP REQ :twitch.tv/tags twitch.tv/commands');
    ircSocket.send(`JOIN #${channel}`);
  };
  ircSocket.onmessage = e => handleIRCMessage(e.data);
  ircSocket.onerror = (e) => { console.error('IRC WebSocket error', e); addChatMsg('system', '', 'WebSocket error — if on GitHub Pages, try enabling insecure content or check F12 console.'); setConnState(false); };
  ircSocket.onclose = (e) => { console.log('IRC closed, code:', e.code, 'reason:', e.reason); 
    setConnState(false);
    addChatMsg('system', '', 'Disconnected. Retrying in 10s…');
    reconnectTimeout = setTimeout(connectIRC, 10000);
  };
}

function handleIRCMessage(raw) {
  const lines = raw.split('\r\n').filter(Boolean);
  lines.forEach(line => {
    if (line.startsWith('PING')) { ircSocket.send('PONG :tmi.twitch.tv'); return; }
    let tags = {}, rest = line;
    if (line.startsWith('@')) {
      const sp = line.indexOf(' ');
      const tagStr = line.substring(1, sp);
      rest = line.substring(sp + 1);
      tagStr.split(';').forEach(t => { const [k,v] = t.split('='); tags[k] = v || ''; });
    }
    const parts = rest.split(' ');
    const prefix = parts[0].startsWith(':') ? parts.shift().slice(1) : '';
    const cmd = parts.shift();
    const params = parts.join(' ');
    if (cmd === '376' || cmd === '001') { setConnState(true); addChatMsg('system', '', `✓ Connected to #${state.streamer.name}`); }
    if (cmd === 'PRIVMSG') {
      const nick = prefix.split('!')[0];
      const display = tags['display-name'] || nick;
      const msgMatch = params.match(/^#\S+ :(.*)/);
      const text = msgMatch ? msgMatch[1] : '';
      addChatMsg('user', display, text);
      checkCommandTrigger(nick, display, text);
    }
    if (cmd === 'NOTICE') { addChatMsg('system', '', params.replace(/^#\S+ :/, '')); }
  });
}

function setConnState(connected) {
  isConnected = connected;
  const dot = document.getElementById('conn-dot');
  const lbl = document.getElementById('conn-label');
  if (connected) { dot.classList.add('connected'); lbl.style.color = 'var(--green)'; lbl.textContent = `#${state.streamer.name}`; }
  else { dot.classList.remove('connected'); lbl.style.color = 'var(--muted)'; lbl.textContent = 'Disconnected'; }
}

document.getElementById('btn-connect-chat').addEventListener('click', () => {
  if (reconnectTimeout) clearTimeout(reconnectTimeout);
  connectIRC();
});

// ─── COMMAND EXECUTION ────────────────────────────────────────────────────────
function checkCommandTrigger(nick, display, text) {
  const msgLower = text.trim().toLowerCase();
  state.commands.forEach(cmd => {
    if (cmd.enabled === false) return;
    const triggers = buildTriggers(cmd).map(t => t.toLowerCase());
    let matched = false;
    let args = '';
    for (const trigger of triggers) {
      if (cmd.wildcards) {
        if (msgLower.startsWith(trigger)) { matched = true; args = text.trim().slice(trigger.length).trim(); break; }
      } else {
        const parts = msgLower.split(/\s+/);
        if (parts[0] === trigger) { matched = true; args = text.trim().split(/\s+/).slice(1).join(' '); break; }
      }
    }
    if (matched) runCommand(cmd, nick, display, args);
  });
}

function runCommand(cmd, nick, display, args) {
  addChatMsg('cmd', '', `▶ Running command: ${cmd.name} (triggered by ${display})`);
  (cmd.actions || []).forEach(action => {
    setTimeout(() => execAction(action, nick, display, args), action.delay || 0);
  });
}

function execAction(action, nick, display, args) {
  if (action.type === 'chat') {
    let msg = (action.message || '').replace('{username}', display).replace('{channel}', state.streamer ? state.streamer.name : '').replace('{args}', args);
    sendChatMessage(msg);
  }
  if (action.type === 'sound') playSound(action);
  if (action.type === 'overlay') showOverlay(action);
}

function sendChatMessage(msg) {
  const sender = state.bot || state.streamer;
  addChatMsg('user', sender ? (sender.display || sender.name) : 'Bot', msg);
  if (ircSocket && ircSocket.readyState === WebSocket.OPEN && state.streamer && !state.streamer.demo) {
    if (sender && sender.token) {
      ircSocket.send(`PRIVMSG #${state.streamer.name} :${msg}`);
    }
  }
}

function playSound(action) {
  const file = state.mediaLib.sounds.find(s => s.id === action.fileId);
  if (!file) return;
  const audio = new Audio(file.b64);
  audio.volume = action.volume !== undefined ? Math.min(1, Math.max(0, action.volume)) : 1;
  audio.play().catch(e => console.warn('Audio play failed:', e));
}

function showOverlay(action) {
  const file = state.mediaLib.overlays.find(o => o.id === action.fileId);
  if (!file) return;
  const container = document.getElementById('overlay-container');
  const el = file.mime.startsWith('video') ? document.createElement('video') : document.createElement('img');
  el.src = file.b64;
  if (el.tagName === 'VIDEO') { el.autoplay = true; el.muted = false; el.loop = action.duration === 0; }
  const pos = action.position || 'center';
  if (action.width) el.style.width = `${action.width}px`;
  if (pos === 'center') Object.assign(el.style, { top: '50%', left: '50%', transform: 'translate(-50%,-50%)' });
  else if (pos === 'top-left') Object.assign(el.style, { top: '20px', left: '20px' });
  else if (pos === 'top-right') Object.assign(el.style, { top: '20px', right: '20px' });
  else if (pos === 'bottom-left') Object.assign(el.style, { bottom: '20px', left: '20px' });
  else if (pos === 'bottom-right') Object.assign(el.style, { bottom: '20px', right: '20px' });
  else if (pos === 'top-center') Object.assign(el.style, { top: '20px', left: '50%', transform: 'translateX(-50%)' });
  else if (pos === 'bottom-center') Object.assign(el.style, { bottom: '20px', left: '50%', transform: 'translateX(-50%)' });
  el.style.borderRadius = '8px';
  el.style.maxWidth = '80vw';
  el.style.maxHeight = '80vh';
  container.appendChild(el);
  const dur = action.duration || 3000;
  if (dur > 0) setTimeout(() => el.remove(), dur);
}

// ─── CHAT LOG ─────────────────────────────────────────────────────────────────
function addChatMsg(type, user, text) {
  const log = document.getElementById('chat-log');
  const div = document.createElement('div');
  div.className = 'chat-msg';
  if (type === 'system') div.innerHTML = `<span class="chat-system">${escHtml(text)}</span>`;
  else if (type === 'cmd') div.innerHTML = `<span class="chat-cmd">${escHtml(text)}</span>`;
  else div.innerHTML = `<span class="chat-user">${escHtml(user)}:</span><span class="chat-text">${escHtml(text)}</span>`;
  log.appendChild(div);
  while (log.children.length > 200) log.removeChild(log.firstChild);
  log.scrollTop = log.scrollHeight;
}
document.getElementById('btn-clear-chat').addEventListener('click', () => { document.getElementById('chat-log').innerHTML = ''; });

// ─── UTILS ────────────────────────────────────────────────────────────────────
function escHtml(s) { return String(s || '').replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;'); }

// ─── BOOT ─────────────────────────────────────────────────────────────────────
loadState();
if (handleOAuthCallback()) {
  // showApp called after user fetch completes
} else if (state.streamer) {
  showApp();
} else {
  showLogin();
}
</script>
</body>
</html>
