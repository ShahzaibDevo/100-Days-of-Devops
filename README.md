
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>DevOps Learning Roadmap</title>
<link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet">
<style>
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  :root {
    --bg: #0b0f1a;
    --bg2: #111827;
    --bg3: #1a2235;
    --surface: #1e2d3d;
    --border: rgba(99,179,237,0.12);
    --border2: rgba(99,179,237,0.22);
    --text: #e2e8f0;
    --muted: #7c8fa6;
    --accent-blue: #63b3ed;
    --accent-cyan: #76e4f7;
    --accent-green: #68d391;
    --accent-orange: #f6ad55;
    --accent-purple: #b794f4;
    --accent-pink: #f687b3;
    --accent-red: #fc8181;
    --accent-teal: #4fd1c5;
  }

  body {
    font-family: 'Space Grotesk', sans-serif;
    background: var(--bg);
    color: var(--text);
    min-height: 100vh;
    line-height: 1.6;
    overflow-x: hidden;
  }

  /* Grid bg pattern */
  body::before {
    content: '';
    position: fixed;
    inset: 0;
    background-image:
      linear-gradient(rgba(99,179,237,0.03) 1px, transparent 1px),
      linear-gradient(90deg, rgba(99,179,237,0.03) 1px, transparent 1px);
    background-size: 40px 40px;
    pointer-events: none;
    z-index: 0;
  }

  .container { max-width: 1200px; margin: 0 auto; padding: 0 2rem; position: relative; z-index: 1; }

  /* HERO */
  .hero {
    padding: 6rem 0 4rem;
    text-align: center;
    position: relative;
  }

  .hero-badge {
    display: inline-flex;
    align-items: center;
    gap: 8px;
    background: rgba(99,179,237,0.08);
    border: 1px solid var(--border2);
    border-radius: 100px;
    padding: 6px 16px;
    font-size: 12px;
    font-family: 'JetBrains Mono', monospace;
    color: var(--accent-cyan);
    letter-spacing: 0.08em;
    margin-bottom: 2rem;
    text-transform: uppercase;
  }

  .hero-badge::before {
    content: '';
    width: 6px; height: 6px;
    background: var(--accent-green);
    border-radius: 50%;
    box-shadow: 0 0 6px var(--accent-green);
  }

  h1 {
    font-size: clamp(2.8rem, 6vw, 5rem);
    font-weight: 700;
    line-height: 1.05;
    letter-spacing: -0.03em;
    margin-bottom: 1.5rem;
  }

  .gradient-text {
    background: linear-gradient(135deg, var(--accent-blue) 0%, var(--accent-cyan) 50%, var(--accent-teal) 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
  }

  .hero-sub {
    font-size: 1.1rem;
    color: var(--muted);
    max-width: 560px;
    margin: 0 auto 3rem;
    font-weight: 300;
  }

  .hero-stats {
    display: flex;
    justify-content: center;
    gap: 3rem;
    flex-wrap: wrap;
  }

  .stat {
    text-align: center;
  }

  .stat-number {
    font-size: 2rem;
    font-weight: 700;
    font-family: 'JetBrains Mono', monospace;
    color: var(--accent-cyan);
  }

  .stat-label {
    font-size: 0.75rem;
    color: var(--muted);
    text-transform: uppercase;
    letter-spacing: 0.1em;
    margin-top: 2px;
  }

  /* SECTION */
  section { padding: 5rem 0; }

  .section-header {
    display: flex;
    align-items: center;
    gap: 1rem;
    margin-bottom: 2.5rem;
  }

  .section-icon {
    width: 42px; height: 42px;
    border-radius: 10px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 1.2rem;
    flex-shrink: 0;
  }

  .section-title {
    font-size: 1.8rem;
    font-weight: 700;
    letter-spacing: -0.02em;
  }

  .section-sub {
    font-size: 0.85rem;
    color: var(--muted);
    font-family: 'JetBrains Mono', monospace;
    margin-top: 2px;
  }

  /* TECH GRID */
  .tech-categories {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
    gap: 1.5rem;
  }

  .tech-card {
    background: var(--bg2);
    border: 1px solid var(--border);
    border-radius: 16px;
    padding: 1.5rem;
    transition: border-color 0.2s, transform 0.2s;
    position: relative;
    overflow: hidden;
  }

  .tech-card::before {
    content: '';
    position: absolute;
    inset: 0;
    opacity: 0;
    transition: opacity 0.3s;
    pointer-events: none;
  }

  .tech-card:hover { transform: translateY(-2px); }
  .tech-card:hover::before { opacity: 1; }

  .tech-card.blue::before { background: radial-gradient(ellipse at top left, rgba(99,179,237,0.06), transparent 60%); }
  .tech-card.cyan::before { background: radial-gradient(ellipse at top left, rgba(118,228,247,0.06), transparent 60%); }
  .tech-card.green::before { background: radial-gradient(ellipse at top left, rgba(104,211,145,0.06), transparent 60%); }
  .tech-card.orange::before { background: radial-gradient(ellipse at top left, rgba(246,173,85,0.06), transparent 60%); }
  .tech-card.purple::before { background: radial-gradient(ellipse at top left, rgba(183,148,244,0.06), transparent 60%); }
  .tech-card.pink::before { background: radial-gradient(ellipse at top left, rgba(246,135,179,0.06), transparent 60%); }
  .tech-card.teal::before { background: radial-gradient(ellipse at top left, rgba(79,209,197,0.06), transparent 60%); }

  .tech-card.blue:hover { border-color: rgba(99,179,237,0.35); }
  .tech-card.cyan:hover { border-color: rgba(118,228,247,0.35); }
  .tech-card.green:hover { border-color: rgba(104,211,145,0.35); }
  .tech-card.orange:hover { border-color: rgba(246,173,85,0.35); }
  .tech-card.purple:hover { border-color: rgba(183,148,244,0.35); }
  .tech-card.pink:hover { border-color: rgba(246,135,179,0.35); }
  .tech-card.teal:hover { border-color: rgba(79,209,197,0.35); }

  .card-header {
    display: flex;
    align-items: center;
    gap: 10px;
    margin-bottom: 1.2rem;
  }

  .card-icon {
    width: 36px; height: 36px;
    border-radius: 8px;
    display: flex; align-items: center; justify-content: center;
    font-size: 1rem;
    flex-shrink: 0;
  }

  .card-title {
    font-size: 1rem;
    font-weight: 600;
    letter-spacing: -0.01em;
  }

  .card-count {
    margin-left: auto;
    font-size: 11px;
    font-family: 'JetBrains Mono', monospace;
    color: var(--muted);
    background: var(--bg3);
    padding: 2px 8px;
    border-radius: 100px;
  }

  .tech-list { display: flex; flex-wrap: wrap; gap: 8px; }

  .tech-pill {
    display: flex;
    align-items: center;
    gap: 6px;
    padding: 5px 12px;
    border-radius: 8px;
    font-size: 12.5px;
    font-weight: 500;
    border: 1px solid transparent;
    cursor: default;
    transition: background 0.15s;
  }

  .tech-pill .dot {
    width: 5px; height: 5px;
    border-radius: 50%;
    flex-shrink: 0;
  }

  /* Color schemes for pills */
  .pill-blue  { background: rgba(99,179,237,0.1); color: var(--accent-blue); border-color: rgba(99,179,237,0.2); }
  .pill-cyan  { background: rgba(118,228,247,0.1); color: var(--accent-cyan); border-color: rgba(118,228,247,0.2); }
  .pill-green { background: rgba(104,211,145,0.1); color: var(--accent-green); border-color: rgba(104,211,145,0.2); }
  .pill-orange{ background: rgba(246,173,85,0.1); color: var(--accent-orange); border-color: rgba(246,173,85,0.2); }
  .pill-purple{ background: rgba(183,148,244,0.1); color: var(--accent-purple); border-color: rgba(183,148,244,0.2); }
  .pill-pink  { background: rgba(246,135,179,0.1); color: var(--accent-pink); border-color: rgba(246,135,179,0.2); }
  .pill-teal  { background: rgba(79,209,197,0.1); color: var(--accent-teal); border-color: rgba(79,209,197,0.2); }
  .pill-red   { background: rgba(252,129,129,0.1); color: var(--accent-red); border-color: rgba(252,129,129,0.2); }

  /* DIVIDER */
  .divider {
    height: 1px;
    background: linear-gradient(90deg, transparent, var(--border2), transparent);
    margin: 0;
  }

  /* LEARNING PATH */
  .timeline {
    position: relative;
    padding-left: 2.5rem;
  }

  .timeline::before {
    content: '';
    position: absolute;
    left: 10px; top: 0; bottom: 0;
    width: 2px;
    background: linear-gradient(to bottom, var(--accent-blue), var(--accent-cyan), var(--accent-teal), var(--accent-green), var(--accent-orange));
    border-radius: 2px;
  }

  .timeline-item {
    position: relative;
    margin-bottom: 2rem;
  }

  .timeline-item::before {
    content: '';
    position: absolute;
    left: -2rem;
    top: 1.2rem;
    width: 12px; height: 12px;
    border-radius: 50%;
    border: 2px solid;
    background: var(--bg);
    z-index: 1;
  }

  .timeline-item:nth-child(1)::before { border-color: var(--accent-blue); box-shadow: 0 0 8px rgba(99,179,237,0.4); }
  .timeline-item:nth-child(2)::before { border-color: var(--accent-cyan); box-shadow: 0 0 8px rgba(118,228,247,0.4); }
  .timeline-item:nth-child(3)::before { border-color: var(--accent-teal); box-shadow: 0 0 8px rgba(79,209,197,0.4); }
  .timeline-item:nth-child(4)::before { border-color: var(--accent-green); box-shadow: 0 0 8px rgba(104,211,145,0.4); }
  .timeline-item:nth-child(5)::before { border-color: var(--accent-orange); box-shadow: 0 0 8px rgba(246,173,85,0.4); }

  .timeline-card {
    background: var(--bg2);
    border: 1px solid var(--border);
    border-radius: 14px;
    padding: 1.5rem;
    transition: border-color 0.2s;
  }

  .timeline-card:hover { border-color: var(--border2); }

  .timeline-meta {
    display: flex;
    align-items: center;
    gap: 12px;
    margin-bottom: 0.75rem;
    flex-wrap: wrap;
  }

  .week-badge {
    font-family: 'JetBrains Mono', monospace;
    font-size: 11px;
    font-weight: 500;
    padding: 3px 10px;
    border-radius: 100px;
    letter-spacing: 0.05em;
    text-transform: uppercase;
  }

  .week-1 { background: rgba(99,179,237,0.12); color: var(--accent-blue); }
  .week-2 { background: rgba(118,228,247,0.12); color: var(--accent-cyan); }
  .week-3 { background: rgba(79,209,197,0.12); color: var(--accent-teal); }
  .week-4 { background: rgba(104,211,145,0.12); color: var(--accent-green); }
  .week-5 { background: rgba(246,173,85,0.12); color: var(--accent-orange); }

  .phase-title {
    font-size: 1.15rem;
    font-weight: 700;
    letter-spacing: -0.01em;
  }

  .phase-desc {
    color: var(--muted);
    font-size: 0.9rem;
    line-height: 1.6;
  }

  .phase-topics {
    display: flex;
    flex-wrap: wrap;
    gap: 6px;
    margin-top: 1rem;
  }

  .topic-tag {
    font-size: 11.5px;
    padding: 3px 10px;
    background: var(--bg3);
    border: 1px solid var(--border);
    border-radius: 6px;
    color: var(--muted);
    font-family: 'JetBrains Mono', monospace;
  }

  /* FOOTER */
  .footer {
    padding: 3rem 0;
    text-align: center;
    border-top: 1px solid var(--border);
    color: var(--muted);
    font-size: 0.85rem;
  }

  .footer-brand {
    font-family: 'JetBrains Mono', monospace;
    color: var(--accent-cyan);
    font-weight: 500;
    margin-bottom: 0.5rem;
    font-size: 1rem;
  }

  /* RESPONSIVE */
  @media (max-width: 640px) {
    .hero { padding: 4rem 0 3rem; }
    .hero-stats { gap: 2rem; }
    .tech-categories { grid-template-columns: 1fr; }
    h1 { font-size: 2.5rem; }
  }

  /* Animated orbs in hero */
  .orb {
    position: absolute;
    border-radius: 50%;
    filter: blur(80px);
    pointer-events: none;
    opacity: 0.12;
  }
  .orb-1 { width: 400px; height: 400px; background: var(--accent-blue); top: -100px; left: -100px; }
  .orb-2 { width: 300px; height: 300px; background: var(--accent-purple); top: -50px; right: 0; }
</style>
</head>
<body>

<!-- HERO -->
<div class="hero" style="position:relative; overflow:hidden;">
  <div class="orb orb-1"></div>
  <div class="orb orb-2"></div>
  <div class="container">
    <div class="hero-badge">DevOps Engineering Roadmap</div>
    <h1>
      From Zero to<br>
      <span class="gradient-text">Production-Ready</span>
    </h1>
    <p class="hero-sub">
      A structured 12-week journey through modern infrastructure, containerization, orchestration, and automation.
    </p>
    <div class="hero-stats">
      <div class="stat">
        <div class="stat-number">12</div>
        <div class="stat-label">Weeks</div>
      </div>
      <div class="stat">
        <div class="stat-number">7</div>
        <div class="stat-label">Categories</div>
      </div>
      <div class="stat">
        <div class="stat-number">25+</div>
        <div class="stat-label">Technologies</div>
      </div>
      <div class="stat">
        <div class="stat-number">5</div>
        <div class="stat-label">Learning Phases</div>
      </div>
    </div>
  </div>
</div>

<div class="divider"></div>

<!-- TECHNOLOGIES COVERED -->
<section>
  <div class="container">
    <div class="section-header">
      <div class="section-icon" style="background:rgba(99,179,237,0.1); border:1px solid rgba(99,179,237,0.2);">🛠️</div>
      <div>
        <div class="section-title">Technologies Covered</div>
        <div class="section-sub">// full stack devops toolkit</div>
      </div>
    </div>

    <div class="tech-categories">

      <!-- Infrastructure & OS -->
      <div class="tech-card blue">
        <div class="card-header">
          <div class="card-icon" style="background:rgba(99,179,237,0.1); border:1px solid rgba(99,179,237,0.18);">🖥️</div>
          <div class="card-title">Infrastructure &amp; OS</div>
          <div class="card-count">4 topics</div>
        </div>
        <div class="tech-list">
          <div class="tech-pill pill-blue"><span class="dot" style="background:var(--accent-blue)"></span>Linux (CentOS, Ubuntu)</div>
          <div class="tech-pill pill-blue"><span class="dot" style="background:var(--accent-blue)"></span>Shell Scripting (Bash)</div>
          <div class="tech-pill pill-blue"><span class="dot" style="background:var(--accent-blue)"></span>System Administration</div>
          <div class="tech-pill pill-blue"><span class="dot" style="background:var(--accent-blue)"></span>Network Configuration</div>
        </div>
        <p style="margin-top:1rem; font-size:0.82rem; color:var(--muted); line-height:1.5;">
          The operating system layer. Master Linux commands, user/permission management, process control, and basic networking — everything runs on top of this.
        </p>
      </div>

      <!-- Containerization -->
      <div class="tech-card cyan">
        <div class="card-header">
          <div class="card-icon" style="background:rgba(118,228,247,0.1); border:1px solid rgba(118,228,247,0.18);">📦</div>
          <div class="card-title">Containerization</div>
          <div class="card-count">3 topics</div>
        </div>
        <div class="tech-list">
          <div class="tech-pill pill-cyan"><span class="dot" style="background:var(--accent-cyan)"></span>Docker</div>
          <div class="tech-pill pill-cyan"><span class="dot" style="background:var(--accent-cyan)"></span>Docker Compose</div>
          <div class="tech-pill pill-cyan"><span class="dot" style="background:var(--accent-cyan)"></span>Container Registry</div>
        </div>
        <p style="margin-top:1rem; font-size:0.82rem; color:var(--muted); line-height:1.5;">
          Package your app and its dependencies into a portable container. Docker isolates environments; Compose runs multi-service apps with a single command; registries store your images.
        </p>
      </div>

      <!-- Orchestration -->
      <div class="tech-card teal">
        <div class="card-header">
          <div class="card-icon" style="background:rgba(79,209,197,0.1); border:1px solid rgba(79,209,197,0.18);">⚙️</div>
          <div class="card-title">Orchestration</div>
          <div class="card-count">3 topics</div>
        </div>
        <div class="tech-list">
          <div class="tech-pill pill-teal"><span class="dot" style="background:var(--accent-teal)"></span>Kubernetes</div>
          <div class="tech-pill pill-teal"><span class="dot" style="background:var(--accent-teal)"></span>kubectl</div>
          <div class="tech-pill pill-teal"><span class="dot" style="background:var(--accent-teal)"></span>YAML</div>
        </div>
        <p style="margin-top:1rem; font-size:0.82rem; color:var(--muted); line-height:1.5;">
          Scale containers across many machines. Kubernetes handles scheduling, self-healing, and load balancing. kubectl is the CLI. YAML describes everything declaratively.
        </p>
      </div>

      <!-- Version Control -->
      <div class="tech-card green">
        <div class="card-header">
          <div class="card-icon" style="background:rgba(104,211,145,0.1); border:1px solid rgba(104,211,145,0.18);">🌿</div>
          <div class="card-title">Version Control</div>
          <div class="card-count">3 topics</div>
        </div>
        <div class="tech-list">
          <div class="tech-pill pill-green"><span class="dot" style="background:var(--accent-green)"></span>Git</div>
          <div class="tech-pill pill-green"><span class="dot" style="background:var(--accent-green)"></span>GitHub</div>
          <div class="tech-pill pill-green"><span class="dot" style="background:var(--accent-green)"></span>Git Hooks</div>
        </div>
        <p style="margin-top:1rem; font-size:0.82rem; color:var(--muted); line-height:1.5;">
          Track every code change. Git is the local version control system; GitHub adds collaboration via PRs and code review; Hooks automate checks on commit or push.
        </p>
      </div>

      <!-- Configuration Management -->
      <div class="tech-card purple">
        <div class="card-header">
          <div class="card-icon" style="background:rgba(183,148,244,0.1); border:1px solid rgba(183,148,244,0.18);">🔧</div>
          <div class="card-title">Configuration Management</div>
          <div class="card-count">2 topics</div>
        </div>
        <div class="tech-list">
          <div class="tech-pill pill-purple"><span class="dot" style="background:var(--accent-purple)"></span>Ansible</div>
          <div class="tech-pill pill-purple"><span class="dot" style="background:var(--accent-purple)"></span>YAML Playbooks</div>
        </div>
        <p style="margin-top:1rem; font-size:0.82rem; color:var(--muted); line-height:1.5;">
          Automate server setup and software deployment without writing custom scripts. Ansible uses playbooks written in YAML to configure hundreds of servers identically.
        </p>
      </div>

      <!-- Web Servers & Databases -->
      <div class="tech-card orange">
        <div class="card-header">
          <div class="card-icon" style="background:rgba(246,173,85,0.1); border:1px solid rgba(246,173,85,0.18);">🗄️</div>
          <div class="card-title">Web Servers &amp; Databases</div>
          <div class="card-count">4 topics</div>
        </div>
        <div class="tech-list">
          <div class="tech-pill pill-orange"><span class="dot" style="background:var(--accent-orange)"></span>Nginx</div>
          <div class="tech-pill pill-orange"><span class="dot" style="background:var(--accent-orange)"></span>Apache</div>
          <div class="tech-pill pill-orange"><span class="dot" style="background:var(--accent-orange)"></span>MySQL / MariaDB</div>
          <div class="tech-pill pill-orange"><span class="dot" style="background:var(--accent-orange)"></span>SSL / Load Balancer</div>
        </div>
        <p style="margin-top:1rem; font-size:0.82rem; color:var(--muted); line-height:1.5;">
          Serve web traffic and persist data. Nginx excels as a reverse proxy and load balancer; Apache powers the LAMP stack; MySQL/MariaDB store structured data.
        </p>
      </div>

      <!-- CI/CD & Automation -->
      <div class="tech-card pink">
        <div class="card-header">
          <div class="card-icon" style="background:rgba(246,135,179,0.1); border:1px solid rgba(246,135,179,0.18);">🚀</div>
          <div class="card-title">CI/CD, Automation &amp; Provisioning</div>
          <div class="card-count">3 topics</div>
        </div>
        <div class="tech-list">
          <div class="tech-pill pill-pink"><span class="dot" style="background:var(--accent-pink)"></span>Terraform</div>
          <div class="tech-pill pill-pink"><span class="dot" style="background:var(--accent-pink)"></span>Jenkins</div>
          <div class="tech-pill pill-pink"><span class="dot" style="background:var(--accent-pink)"></span>Ansible Playbook</div>
        </div>
        <p style="margin-top:1rem; font-size:0.82rem; color:var(--muted); line-height:1.5;">
          Terraform provisions cloud resources as code; Jenkins automates build→test→deploy pipelines on every commit; Ansible playbooks keep infrastructure config drift-free.
        </p>
      </div>

    </div>
  </div>
</section>

<div class="divider"></div>

<!-- LEARNING PATH -->
<section>
  <div class="container">
    <div class="section-header">
      <div class="section-icon" style="background:rgba(104,211,145,0.1); border:1px solid rgba(104,211,145,0.2);">🎯</div>
      <div>
        <div class="section-title">Learning Path</div>
        <div class="section-sub">// 12-week structured curriculum</div>
      </div>
    </div>

    <div class="timeline">

      <div class="timeline-item">
        <div class="timeline-card">
          <div class="timeline-meta">
            <span class="week-badge week-1">Week 1 – 3</span>
            <span style="font-size:11px; color:var(--muted); font-family:'JetBrains Mono',monospace;">Phase 01</span>
          </div>
          <div class="phase-title">Linux Fundamentals</div>
          <p class="phase-desc" style="margin-top:0.4rem;">
            The foundation of everything. Learn how Linux manages users, files, processes, and the network. You'll write Bash scripts to automate repetitive tasks and gain comfort with the command line — a skill every other phase builds on.
          </p>
          <div class="phase-topics">
            <span class="topic-tag">user-management</span>
            <span class="topic-tag">file-permissions</span>
            <span class="topic-tag">bash-scripting</span>
            <span class="topic-tag">process-control</span>
            <span class="topic-tag">networking-basics</span>
            <span class="topic-tag">systemd</span>
            <span class="topic-tag">cron-jobs</span>
          </div>
        </div>
      </div>

      <div class="timeline-item">
        <div class="timeline-card">
          <div class="timeline-meta">
            <span class="week-badge week-2">Week 4 – 5</span>
            <span style="font-size:11px; color:var(--muted); font-family:'JetBrains Mono',monospace;">Phase 02</span>
          </div>
          <div class="phase-title">Version Control</div>
          <p class="phase-desc" style="margin-top:0.4rem;">
            Learn how professional teams collaborate on code. Git gives you a full history of every change; GitHub connects you to teammates. You'll practice branching strategies, resolving merge conflicts, rebasing, and automating checks with hooks.
          </p>
          <div class="phase-topics">
            <span class="topic-tag">git-branching</span>
            <span class="topic-tag">merging</span>
            <span class="topic-tag">rebasing</span>
            <span class="topic-tag">pull-requests</span>
            <span class="topic-tag">git-hooks</span>
            <span class="topic-tag">code-review</span>
          </div>
        </div>
      </div>

      <div class="timeline-item">
        <div class="timeline-card">
          <div class="timeline-meta">
            <span class="week-badge week-3">Week 6 – 7</span>
            <span style="font-size:11px; color:var(--muted); font-family:'JetBrains Mono',monospace;">Phase 03</span>
          </div>
          <div class="phase-title">Containerization</div>
          <p class="phase-desc" style="margin-top:0.4rem;">
            Package applications so they run the same everywhere. You'll build Docker images, manage container lifecycles, define multi-service apps with Compose, and push images to a registry — the step before Kubernetes.
          </p>
          <div class="phase-topics">
            <span class="topic-tag">dockerfile</span>
            <span class="topic-tag">images-layers</span>
            <span class="topic-tag">volumes</span>
            <span class="topic-tag">networking</span>
            <span class="topic-tag">docker-compose</span>
            <span class="topic-tag">registry-push</span>
          </div>
        </div>
      </div>

      <div class="timeline-item">
        <div class="timeline-card">
          <div class="timeline-meta">
            <span class="week-badge week-4">Week 8 – 10</span>
            <span style="font-size:11px; color:var(--muted); font-family:'JetBrains Mono',monospace;">Phase 04</span>
          </div>
          <div class="phase-title">Kubernetes</div>
          <p class="phase-desc" style="margin-top:0.4rem;">
            Scale containers across a cluster automatically. You'll deploy apps using Pods, Deployments, and Services; manage persistent storage with Volumes; and use ReplicaSets for high availability — all through kubectl and YAML manifests.
          </p>
          <div class="phase-topics">
            <span class="topic-tag">pods</span>
            <span class="topic-tag">deployments</span>
            <span class="topic-tag">services</span>
            <span class="topic-tag">replica-sets</span>
            <span class="topic-tag">volumes-pvc</span>
            <span class="topic-tag">namespaces</span>
            <span class="topic-tag">ingress</span>
            <span class="topic-tag">kubectl</span>
          </div>
        </div>
      </div>

      <div class="timeline-item">
        <div class="timeline-card">
          <div class="timeline-meta">
            <span class="week-badge week-5">Week 10 – 12</span>
            <span style="font-size:11px; color:var(--muted); font-family:'JetBrains Mono',monospace;">Phase 05</span>
          </div>
          <div class="phase-title">Jenkins, Terraform &amp; Ansible</div>
          <p class="phase-desc" style="margin-top:0.4rem;">
            Bring everything together with full automation. Terraform provisions cloud resources declaratively; Jenkins builds and deploys on every code push; Ansible playbooks configure servers without manual SSH — the full DevOps feedback loop.
          </p>
          <div class="phase-topics">
            <span class="topic-tag">jenkins-pipelines</span>
            <span class="topic-tag">terraform-plan-apply</span>
            <span class="topic-tag">ansible-roles</span>
            <span class="topic-tag">iac-modules</span>
            <span class="topic-tag">secrets-mgmt</span>
            <span class="topic-tag">cloud-provisioning</span>
            <span class="topic-tag">blue-green-deploy</span>
          </div>
        </div>
      </div>

    </div>
  </div>
</section>

<!-- FOOTER -->
<footer class="footer">
  <div class="container">
    <div class="footer-brand">$ devops --roadmap complete ✓</div>
    <p>Build. Ship. Scale. Repeat.</p>
  </div>
</footer>

</body>
</html>
