import React, { useEffect, useMemo, useState } from 'react';

const STORAGE_KEY = 'gaa-football-match-v1';

const createInitialState = () => ({
  match: {
    teamA: 'London GAA',
    teamB: 'Opposition',
    isStarted: false,
    startedAt: null,
  },
  teams: {
    A: {
      goals: 0,
      points: 0,
      tackles: 0,
      attacks: { total: 0, shots: 0, scores: 0, wides: 0, dropShort: 0, recycled: 0, lostPossession: 0 },
      kickouts: { total: 0, long: 0, short: 0, won: 0, lost: 0 },
    },
    B: {
      goals: 0,
      points: 0,
      tackles: 0,
      attacks: { total: 0, shots: 0, scores: 0, wides: 0, dropShort: 0, recycled: 0, lostPossession: 0 },
      kickouts: { total: 0, long: 0, short: 0, won: 0, lost: 0 },
    },
  },
  events: [],
});

function loadState() {
  try {
    const raw = window.localStorage.getItem(STORAGE_KEY);
    return raw ? JSON.parse(raw) : createInitialState();
  } catch {
    return createInitialState();
  }
}

function percent(numerator, denominator) {
  if (!denominator) return '0%';
  return `${Math.round((numerator / denominator) * 100)}%`;
}

function scoreString(team) {
  return `${team.goals}-${team.points}`;
}

function totalPoints(team) {
  return team.goals * 3 + team.points;
}

function cloneState(state) {
  return JSON.parse(JSON.stringify(state));
}

function App() {
  const [state, setState] = useState(() => loadState());
  const [screen, setScreen] = useState('dashboard');
  const [kickoutFlow, setKickoutFlow] = useState({ open: false, team: 'A', distance: 'long', winner: 'A' });
  const [attackFlow, setAttackFlow] = useState({
    open: false,
    team: 'A',
    result: 'shot',
    shotOutcome: 'score',
    scoreType: 'point',
  });

  useEffect(() => {
    window.localStorage.setItem(STORAGE_KEY, JSON.stringify(state));
  }, [state]);

  const labels = {
    A: state.match.teamA || 'Team A',
    B: state.match.teamB || 'Team B',
  };

  const summary = useMemo(() => {
    const teamA = state.teams.A;
    const teamB = state.teams.B;
    return {
      A: {
        shotRate: percent(teamA.attacks.shots, teamA.attacks.total),
        scoreRate: percent(teamA.attacks.scores, teamA.attacks.shots),
        kickoutWinRate: percent(teamA.kickouts.won, teamA.kickouts.total),
      },
      B: {
        shotRate: percent(teamB.attacks.shots, teamB.attacks.total),
        scoreRate: percent(teamB.attacks.scores, teamB.attacks.shots),
        kickoutWinRate: percent(teamB.kickouts.won, teamB.kickouts.total),
      },
    };
  }, [state]);

  function updateState(mutator) {
    setState((current) => {
      const next = cloneState(current);
      mutator(next);
      return next;
    });
  }

  function pushEvent(next, event) {
    next.events.unshift({ id: crypto.randomUUID(), time: new Date().toISOString(), ...event });
    next.events = next.events.slice(0, 25);
  }

  function adjustCounter(teamKey, field, amount) {
    updateState((next) => {
      next.teams[teamKey][field] = Math.max(0, next.teams[teamKey][field] + amount);
      pushEvent(next, { type: 'counter', team: teamKey, field, amount });
    });
  }

  function adjustScore(teamKey, kind, amount) {
    updateState((next) => {
      next.teams[teamKey][kind] = Math.max(0, next.teams[teamKey][kind] + amount);
      pushEvent(next, { type: 'score', team: teamKey, kind, amount });
    });
  }

  function saveMatchSetup(event) {
    event.preventDefault();
    const form = new FormData(event.currentTarget);
    const teamA = form.get('teamA')?.toString().trim() || 'Team A';
    const teamB = form.get('teamB')?.toString().trim() || 'Team B';
    updateState((next) => {
      next.match.teamA = teamA;
      next.match.teamB = teamB;
      next.match.isStarted = true;
      next.match.startedAt = next.match.startedAt || new Date().toISOString();
    });
    setScreen('dashboard');
  }

  function addKickout() {
    updateState((next) => {
      const { team, distance, winner } = kickoutFlow;
      const kickingTeam = next.teams[team];
      kickingTeam.kickouts.total += 1;
      kickingTeam.kickouts[distance] += 1;
      if (winner === team) kickingTeam.kickouts.won += 1;
      else kickingTeam.kickouts.lost += 1;
      pushEvent(next, {
        type: 'kickout',
        team,
        distance,
        winner,
      });
    });
    setKickoutFlow({ open: false, team: 'A', distance: 'long', winner: 'A' });
    setScreen('dashboard');
  }

  function addAttack() {
    updateState((next) => {
      const { team, result, shotOutcome, scoreType } = attackFlow;
      const attackTeam = next.teams[team];
      attackTeam.attacks.total += 1;

      if (result === 'lostPossession') attackTeam.attacks.lostPossession += 1;
      if (result === 'recycle') attackTeam.attacks.recycled += 1;

      if (result === 'shot') {
        attackTeam.attacks.shots += 1;

        if (shotOutcome === 'wide') attackTeam.attacks.wides += 1;
        if (shotOutcome === 'dropShort') attackTeam.attacks.dropShort += 1;
        if (shotOutcome === 'score') {
          attackTeam.attacks.scores += 1;
          if (scoreType === 'goal') attackTeam.goals += 1;
          if (scoreType === 'point') attackTeam.points += 1;
        }
      }

      pushEvent(next, {
        type: 'attack',
        team,
        result,
        shotOutcome: result === 'shot' ? shotOutcome : null,
        scoreType: result === 'shot' && shotOutcome === 'score' ? scoreType : null,
      });
    });
    setAttackFlow({ open: false, team: 'A', result: 'shot', shotOutcome: 'score', scoreType: 'point' });
    setScreen('dashboard');
  }

  function exportSummary() {
    const a = state.teams.A;
    const b = state.teams.B;
    const lines = [
      `${labels.A} vs ${labels.B}`,
      state.match.startedAt ? `Started: ${new Date(state.match.startedAt).toLocaleString()}` : '',
      '',
      `${labels.A}`,
      `Score: ${scoreString(a)} (${totalPoints(a)})`,
      `Tackles: ${a.tackles}`,
      `Attacks: ${a.attacks.total}`,
      `Shots: ${a.attacks.shots} (${summary.A.shotRate} of attacks)`,
      `Scores from shots: ${a.attacks.scores} (${summary.A.scoreRate} of shots)`,
      `Wides: ${a.attacks.wides}`,
      `Drop short: ${a.attacks.dropShort}`,
      `Recycled: ${a.attacks.recycled}`,
      `Lost possession: ${a.attacks.lostPossession}`,
      `Kickouts: ${a.kickouts.total}`,
      `Kickout wins: ${a.kickouts.won} (${summary.A.kickoutWinRate})`,
      '',
      `${labels.B}`,
      `Score: ${scoreString(b)} (${totalPoints(b)})`,
      `Tackles: ${b.tackles}`,
      `Attacks: ${b.attacks.total}`,
      `Shots: ${b.attacks.shots} (${summary.B.shotRate} of attacks)`,
      `Scores from shots: ${b.attacks.scores} (${summary.B.scoreRate} of shots)`,
      `Wides: ${b.attacks.wides}`,
      `Drop short: ${b.attacks.dropShort}`,
      `Recycled: ${b.attacks.recycled}`,
      `Lost possession: ${b.attacks.lostPossession}`,
      `Kickouts: ${b.kickouts.total}`,
      `Kickout wins: ${b.kickouts.won} (${summary.B.kickoutWinRate})`,
    ].filter(Boolean);

    const text = lines.join('\n');
    if (navigator.clipboard?.writeText) {
      navigator.clipboard.writeText(text);
      alert('Match summary copied to clipboard.');
    } else {
      alert(text);
    }
  }

  function resetMatch() {
    const confirmed = window.confirm('Reset the current match? This will clear all live data.');
    if (!confirmed) return;
    setState(createInitialState());
    setScreen('setup');
  }

  const teamCards = ['A', 'B'].map((teamKey) => {
    const team = state.teams[teamKey];
    return (
      <section className="card" key={teamKey}>
        <div className="team-header">
          <div>
            <h2>{labels[teamKey]}</h2>
            <div className="score-line">
              <span className="score-main">{scoreString(team)}</span>
              <span className="score-total">{totalPoints(team)} total</span>
            </div>
          </div>
        </div>

        <div className="counter-grid">
          <StatCounter label="Goals" value={team.goals} onMinus={() => adjustScore(teamKey, 'goals', -1)} onPlus={() => adjustScore(teamKey, 'goals', 1)} />
          <StatCounter label="Points" value={team.points} onMinus={() => adjustScore(teamKey, 'points', -1)} onPlus={() => adjustScore(teamKey, 'points', 1)} />
          <StatCounter label="Tackles" value={team.tackles} onMinus={() => adjustCounter(teamKey, 'tackles', -1)} onPlus={() => adjustCounter(teamKey, 'tackles', 1)} />
          <MetricCard label="Attacks" value={team.attacks.total} />
          <MetricCard label="Shots from attacks" value={`${team.attacks.shots} · ${team.attacks.total ? summary[teamKey].shotRate : '0%'}`} />
          <MetricCard label="Scores from shots" value={`${team.attacks.scores} · ${team.attacks.shots ? summary[teamKey].scoreRate : '0%'}`} />
          <MetricCard label="Kickouts won" value={`${team.kickouts.won}/${team.kickouts.total} · ${summary[teamKey].kickoutWinRate}`} />
          <MetricCard label="Long / Short KO" value={`${team.kickouts.long} / ${team.kickouts.short}`} />
        </div>

        <div className="detail-grid">
          <SmallMetric label="Wides" value={team.attacks.wides} />
          <SmallMetric label="Drop short" value={team.attacks.dropShort} />
          <SmallMetric label="Recycle" value={team.attacks.recycled} />
          <SmallMetric label="Lost poss." value={team.attacks.lostPossession} />
        </div>
      </section>
    );
  });

  return (
    <div className="app-shell">
      <header className="app-header">
        <div>
          <p className="eyebrow">Gaelic football match tracker</p>
          <h1>{labels.A} vs {labels.B}</h1>
        </div>
        <div className="header-actions">
          <button className="secondary" onClick={() => setScreen('setup')}>Match setup</button>
          <button className="danger" onClick={resetMatch}>Reset</button>
        </div>
      </header>

      {!state.match.isStarted || screen === 'setup' ? (
        <section className="card form-card">
          <h2>Match setup</h2>
          <form onSubmit={saveMatchSetup} className="setup-form">
            <label>
              Team A
              <input name="teamA" defaultValue={state.match.teamA} placeholder="Enter Team A" />
            </label>
            <label>
              Team B
              <input name="teamB" defaultValue={state.match.teamB} placeholder="Enter Team B" />
            </label>
            <button type="submit">Start match</button>
          </form>
        </section>
      ) : (
        <>
          <nav className="action-bar">
            <button onClick={() => setKickoutFlow((v) => ({ ...v, open: true }))}>Log kickout</button>
            <button onClick={() => setAttackFlow((v) => ({ ...v, open: true }))}>Log attack</button>
            <button onClick={exportSummary}>Copy summary</button>
          </nav>

          <main className="main-grid">{teamCards}</main>

          <section className="card">
            <h2>Recent events</h2>
            <div className="event-list">
              {state.events.length === 0 ? (
                <p className="muted">No match events yet.</p>
              ) : (
                state.events.map((event) => (
                  <div className="event-item" key={event.id}>
                    <span>{renderEvent(event, labels)}</span>
                  </div>
                ))
              )}
            </div>
          </section>
        </>
      )}

      {kickoutFlow.open && (
        <Modal title="Log kickout" onClose={() => setKickoutFlow((v) => ({ ...v, open: false }))}>
          <Field label="Which team took the kickout?">
            <TogglePair value={kickoutFlow.team} onChange={(value) => setKickoutFlow((v) => ({ ...v, team: value }))} labels={labels} />
          </Field>
          <Field label="Distance">
            <ChoiceRow options={[['long', 'Long'], ['short', 'Short']]} value={kickoutFlow.distance} onChange={(value) => setKickoutFlow((v) => ({ ...v, distance: value }))} />
          </Field>
          <Field label="Who won possession?">
            <TogglePair value={kickoutFlow.winner} onChange={(value) => setKickoutFlow((v) => ({ ...v, winner: value }))} labels={labels} />
          </Field>
          <button onClick={addKickout}>Save kickout</button>
        </Modal>
      )}

      {attackFlow.open && (
        <Modal title="Log attack into opposition 45" onClose={() => setAttackFlow((v) => ({ ...v, open: false }))}>
          <Field label="Which team attacked?">
            <TogglePair value={attackFlow.team} onChange={(value) => setAttackFlow((v) => ({ ...v, team: value }))} labels={labels} />
          </Field>
          <Field label="Attack result">
            <ChoiceRow
              options={[
                ['lostPossession', 'Lost possession'],
                ['shot', 'Shot'],
                ['recycle', 'Recycle'],
              ]}
              value={attackFlow.result}
              onChange={(value) => setAttackFlow((v) => ({ ...v, result: value }))}
            />
          </Field>
          {attackFlow.result === 'shot' && (
            <>
              <Field label="Shot outcome">
                <ChoiceRow
                  options={[
                    ['score', 'Score'],
                    ['wide', 'Wide'],
                    ['dropShort', 'Drop short'],
                  ]}
                  value={attackFlow.shotOutcome}
                  onChange={(value) => setAttackFlow((v) => ({ ...v, shotOutcome: value }))}
                />
              </Field>
              {attackFlow.shotOutcome === 'score' && (
                <Field label="Score type">
                  <ChoiceRow
                    options={[
                      ['point', 'Point'],
                      ['goal', 'Goal'],
                    ]}
                    value={attackFlow.scoreType}
                    onChange={(value) => setAttackFlow((v) => ({ ...v, scoreType: value }))}
                  />
                </Field>
              )}
            </>
          )}
          <button onClick={addAttack}>Save attack</button>
        </Modal>
      )}
    </div>
  );
}

function renderEvent(event, labels) {
  const teamLabel = labels[event.team] || event.team;
  if (event.type === 'kickout') return `${teamLabel} kickout · ${event.distance} · won by ${labels[event.winner]}`;
  if (event.type === 'attack') {
    if (event.result !== 'shot') return `${teamLabel} attack · ${event.result}`;
    if (event.shotOutcome !== 'score') return `${teamLabel} attack · shot · ${event.shotOutcome}`;
    return `${teamLabel} attack · shot scored · ${event.scoreType}`;
  }
  if (event.type === 'score') return `${teamLabel} ${event.kind} ${event.amount > 0 ? '+1' : '-1'}`;
  if (event.type === 'counter') return `${teamLabel} ${event.field} ${event.amount > 0 ? '+1' : '-1'}`;
  return 'Event logged';
}

function StatCounter({ label, value, onMinus, onPlus }) {
  return (
    <div className="stat-card">
      <span className="stat-label">{label}</span>
      <div className="stat-controls">
        <button className="mini danger" onClick={onMinus}>−</button>
        <strong>{value}</strong>
        <button className="mini" onClick={onPlus}>+</button>
      </div>
    </div>
  );
}

function MetricCard({ label, value }) {
  return (
    <div className="stat-card muted-card">
      <span className="stat-label">{label}</span>
      <strong>{value}</strong>
    </div>
  );
}

function SmallMetric({ label, value }) {
  return (
    <div className="small-metric">
      <span>{label}</span>
      <strong>{value}</strong>
    </div>
  );
}

function Modal({ title, children, onClose }) {
  return (
    <div className="modal-backdrop" onClick={onClose}>
      <div className="modal-card" onClick={(event) => event.stopPropagation()}>
        <div className="modal-head">
          <h2>{title}</h2>
          <button className="secondary" onClick={onClose}>Close</button>
        </div>
        <div className="modal-body">{children}</div>
      </div>
    </div>
  );
}

function Field({ label, children }) {
  return (
    <div className="field">
      <p className="field-label">{label}</p>
      {children}
    </div>
  );
}

function TogglePair({ value, onChange, labels }) {
  return (
    <div className="choice-row">
      {['A', 'B'].map((key) => (
        <button key={key} className={value === key ? 'active' : 'secondary'} onClick={() => onChange(key)}>
          {labels[key]}
        </button>
      ))}
    </div>
  );
}

function ChoiceRow({ options, value, onChange }) {
  return (
    <div className="choice-row wrap">
      {options.map(([optionValue, label]) => (
        <button key={optionValue} className={value === optionValue ? 'active' : 'secondary'} onClick={() => onChange(optionValue)}>
          {label}
        </button>
      ))}
    </div>
  );
}

export default App;
