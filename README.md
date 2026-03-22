# Hollow-realm
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  <link rel="apple-touch-icon" href="https://i.imgur.com/your-image-here.png">
  <title>The Hollow Realm</title>
  
  <script src="https://unpkg.com/react@18/umd/react.production.min.js crossorigin"></script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js crossorigin"></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
</head>
<body style="margin: 0; background-color: #02040a; -webkit-tap-highlight-color: transparent;">
  <div id="root"></div>

  <script type="text/babel">
    const { useState, useEffect, useRef } = React;

    // ═══════════════════════════════════════════════════
    // CUSTOM HOOKS (DATA PERSISTENCE)
    // ═══════════════════════════════════════════════════
    function useLocalStorage(key, initialValue) {
      const [storedValue, setStoredValue] = useState(() => {
        try {
          const item = window.localStorage.getItem(key);
          return item ? JSON.parse(item) : initialValue;
        } catch (error) {
          return initialValue;
        }
      });

      const setValue = (value) => {
        try {
          const valueToStore = value instanceof Function ? value(storedValue) : value;
          setStoredValue(valueToStore);
          window.localStorage.setItem(key, JSON.stringify(valueToStore));
        } catch (error) {}
      };
      return [storedValue, setValue];
    }

    // ═══════════════════════════════════════════════════
    // GAME DATA
    // ═══════════════════════════════════════════════════
    const NAMES = ["Bubbles","Wiggles","Dottie","Fern","Pippa","Coco","Boo","Thistle","Mochi","Remy","Pebble","Snoot","Clover","Noodle","Zazu","Binky","Tangle","Marshy","Grim","Wick","Wren","Fizz","Dusk","Vex","Rue","Hollow","Sable","Thorn","Cinder","Ash"];
    const ORIGINS = ["The Dark Enchanted Forest","The Crumbling Kingdom","The Underground Caverns","The Hollow's Edge","The Sunken Library"];
    const CLASSES = ["Warrior","Wizard","Rogue","Healer","Merchant","Bard"];
    const CLASS_HP = {Warrior:12,Wizard:7,Rogue:8,Healer:9,Merchant:8,Bard:8};
    const CLASS_ICON = {Warrior:"⚔️",Wizard:"🔮",Rogue:"🗡️",Healer:"💚",Merchant:"💰",Bard:"🎵"};
    const STATUSES = ["Active & Healthy", "Injured", "Recovering", "Missing", "Captured", "Corrupted"];
    const FACTIONS = ["Unaligned", "Moon Squad", "The Hollow Warden's Court", "The Edge Runners", "The Sunken Scholars", "Garrison Exiles"];

    const GIFT_TIERS = [
      { category: "Action Rolls (Timeline Impact)", items: [ { name: "⚔️ Minor Action Roll", coins: "100", cad: "~$1.40", you: "~$0.70", desc: "Small story ripple." }, { name: "⚔️⚔️ Moderate Roll", coins: "500", cad: "~$7.00", you: "~$3.50", desc: "Real consequence." }, { name: "⚔️⚔️⚔️ Major Roll", coins: "5,000", cad: "~$70.00", you: "~$35.00", desc: "Session-changing." }, { name: "⚡ Epic — Lion", coins: "29,999", cad: "~$420.00", you: "~$210.00", desc: "World-altering." } ] },
      { category: "Character Lifecycle", items: [ { name: "🌑 Character Entry", coins: "1,000", cad: "~$14.00", you: "~$7.00", desc: "Permanent creation." }, { name: "⚗️ Resurrection", coins: "6,000", cad: "~$84.00", you: "~$42.00", desc: "Full stats restored." }, { name: "💀 New Character", coins: "1,000", cad: "~$14.00", you: "~$7.00", desc: "Fresh start after death." }, { name: "🛡️ Protection Mode", coins: "1,000", cad: "~$14.00", you: "~$7.00", desc: "Insurance per session." } ] }
    ];

    function d20() { return Math.floor(Math.random() * 20) + 1; }

    // ═══════════════════════════════════════════════════
    // STYLES
    // ═══════════════════════════════════════════════════
    const C = { bg: "#02040a", surface: "rgba(13, 17, 30, 0.45)", card: "rgba(20, 25, 45, 0.5)", border: "rgba(125, 150, 255, 0.15)", borderGlow: "rgba(125, 170, 255, 0.4)", accent: "#818cf8", gold: "#fbbf24", red: "#f87171", green: "#34d399", text: "#f8fafc", dim: "#94a3b8" };

    const S = {
      app: { fontFamily: "-apple-system, BlinkMacSystemFont, sans-serif", backgroundColor: C.bg, color: C.text, minHeight: "100vh", maxWidth: 500, margin: "0 auto", position: "relative", overflowX: "hidden" },
      headerImage: { width: "100%", height: "220px", objectFit: "cover", objectPosition: "center top", maskImage: "linear-gradient(to bottom, black 40%, transparent 100%)", WebkitMaskImage: "linear-gradient(to bottom, black 40%, transparent 100%)", position: "absolute", top: 0, left: 0, zIndex: 0 },
      headerGlass: { background: "rgba(2, 4, 10, 0.3)", backdropFilter: "blur(12px)", WebkitBackdropFilter: "blur(12px)", padding: "20px 16px 16px", position: "sticky", top: 0, zIndex: 100, borderBottom: `1px solid ${C.border}` },
      tabs: { display: "flex", background: "rgba(10, 15, 30, 0.6)", backdropFilter: "blur(10px)", borderBottom: `1px solid ${C.border}`, overflowX: "auto", scrollbarWidth: "none", padding: "4px 8px" },
      tab: (active) => ({ padding: "10px 16px", fontSize: "0.75rem", fontWeight: "600", color: active ? "#fff" : C.dim, cursor: "pointer", whiteSpace: "nowrap", background: active ? "rgba(129, 140, 248, 0.15)" : "transparent", borderRadius: "20px", margin: "4px", flexShrink: 0, border: active ? `1px solid rgba(129, 140, 248, 0.3)` : `1px solid transparent` }),
      panel: { padding: "16px", position: "relative", zIndex: 10, paddingBottom: "100px" },
      card: { background: C.card, backdropFilter: "blur(16px)", WebkitBackdropFilter: "blur(16px)", border: `1px solid ${C.border}`, borderRadius: "16px", padding: "18px", marginBottom: "16px", boxShadow: "0 8px 32px rgba(0, 0, 0, 0.3)" },
      sectionTitle: { fontSize: "0.65rem", fontWeight: "700", textTransform: "uppercase", letterSpacing: "2px", color: C.accent, marginBottom: "12px" },
      input: { width: "100%", padding: "12px 14px", background: "rgba(0, 0, 0, 0.3)", border: `1px solid ${C.border}`, borderRadius: "10px", color: C.text, fontSize: "0.9rem", outline: "none", boxSizing: "border-box" },
      label: { display: "block", fontSize: "0.7rem", fontWeight: "500", color: C.dim, marginBottom: "6px" },
      btn: (bg = "rgba(129, 140, 248, 0.2)", color = C.accent, border = C.accent) => ({ padding: "12px 20px", borderRadius: "12px", border: `1px solid ${border}`, cursor: "pointer", fontSize: "0.85rem", fontWeight: "600", background: bg, color: color, textAlign: "center", width: "100%", boxSizing: "border-box" }),
      btnSm: (bg = "rgba(255,255,255,0.05)", color = "#fff", border = C.border) => ({ padding: "6px 12px", borderRadius: "8px", border: `1px solid ${border}`, cursor: "pointer", fontSize: "0.75rem", fontWeight: "500", background: bg, color: color, display:"flex", alignItems:"center", justifyContent:"center" }),
      tag: (color, border) => ({ fontSize: "0.65rem", padding: "4px 8px", borderRadius: "6px", fontWeight: "600", background: "rgba(0,0,0,0.3)", border: `1px solid ${border}`, color: color, display: "inline-block" })
    };

    // ═══════════════════════════════════════════════════
    // COMPONENTS
    // ═══════════════════════════════════════════════════
    const PlayerCard = ({ p, onAdjHP, onSelect }) => {
      const pct = Math.max(0, (p.hp / p.maxHp) * 100);
      const hpColor = pct > 60 ? C.green : pct > 30 ? C.gold : C.red;

      return (
        <div style={{...S.card, border: p.protectionBalance > 0 ? `1px solid ${C.gold}` : `1px solid ${C.border}`, opacity: p.isDead ? 0.6 : 1, padding:"14px" }}>
          <div style={{ display:"flex", justifyContent:"space-between", alignItems:"flex-start" }}>
            <div>
              <div style={{ fontWeight:"700", fontSize:"1.1rem", color:"#fff" }}>{p.charname} {p.isDead ? "☠️" : ""}</div>
              <div style={{ fontSize:"0.75rem", color:C.dim, marginTop:"2px" }}>@{p.username} · {CLASS_ICON[p.cls]} {p.cls}</div>
            </div>
            <button style={S.btnSm(undefined, undefined, "transparent")} onClick={() => onSelect(p.id)}>⚙️ Edit</button>
          </div>
          
          <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: "8px", marginTop: "10px" }}>
            <div style={{ fontSize:"0.7rem", color:C.dim }}><span style={{color:C.accent}}>Status:</span> {p.status}</div>
            <div style={{ fontSize:"0.7rem", color:C.dim }}><span style={{color:C.accent}}>Faction:</span> {p.faction}</div>
          </div>

          <div style={{ marginTop:"14px", display:"flex", alignItems:"center", gap:"10px" }}>
            <button style={S.btnSm("rgba(248, 113, 113, 0.15)", C.red, "rgba(248, 113, 113, 0.3)")} onClick={() => onAdjHP(p.id, -1, p.charname)}>-1</button>
            <div style={{ flex: 1 }}>
              <div style={{ display:"flex", justifyContent:"space-between", fontSize:"0.65rem", color:C.dim, marginBottom:"4px", fontWeight:"600" }}>
                <span>HP</span><span>{p.hp} / {p.maxHp}</span>
              </div>
              <div style={{ height:"6px", background:"rgba(0,0,0,0.5)", borderRadius:"3px", overflow:"hidden" }}>
                <div style={{ height:"100%", width:`${pct}%`, background:hpColor, borderRadius:"3px", transition:"width 0.3s ease" }} />
              </div>
            </div>
            <button style={S.btnSm("rgba(52, 211, 153, 0.15)", C.green, "rgba(52, 211, 153, 0.3)")} onClick={() => onAdjHP(p.id, 1, p.charname)}>+1</button>
          </div>
        </div>
      );
    };

    // ═══════════════════════════════════════════════════
    // MAIN APP
    // ═══════════════════════════════════════════════════
    function App() {
      const [players, setPlayers] = useLocalStorage("hollow_realm_players", []);
      const [world, setWorld] = useLocalStorage("hollow_realm_world", { arcTitle:"", arcVillain:"", arcThread:"" });
      const [actionLog, setActionLog] = useLocalStorage("hollow_realm_logs", []);
      const [battleHistory, setBattleHistory] = useLocalStorage("hollow_realm_battles", []);
      
      const [tab, setTab] = useState("players");
      const [search, setSearch] = useState("");
      const [selectedPlayer, setSelectedPlayer] = useState(null);
      const [creation, setCreation] = useState({ username:"", charname:"", cls:"", faction:"Unaligned", status:"Active & Healthy" });
      
      const [diceRoll, setDiceRoll] = useState(null);
      const [diceAnim, setDiceAnim] = useState(false);
      
      const [battleP1, setBattleP1] = useState(""); 
      const [battleP2, setBattleP2] = useState("");
      const [coins1, setCoins1] = useState(""); 
      const [coins2, setCoins2] = useState("");
      
      const [ripples, setRipples] = useState([]);
      
      // We'll use a placeholder for the hero image since it's hardcoded in the single file
      // INSTRUCTION: Replace this URL with your uploaded image URL later.
      const HERO_IMAGE_URL = "image.png";

      const handleTouch = (e) => {
        const rect = e.currentTarget.getBoundingClientRect();
        const x = e.clientX - rect.left;
        const y = e.clientY - rect.top;
        const newRipple = { id: Date.now(), x, y };
        setRipples(prev => [...prev, newRipple]);
        setTimeout(() => { setRipples(prev => prev.filter(r => r.id !== newRipple.id)); }, 800);
      };

      const logEvent = (message, type = "info") => {
        const newLog = { id: Date.now(), time: new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'}), message, type };
        setActionLog(prev => [newLog, ...prev].slice(0, 50)); 
      };

      const addPlayer = () => {
        if (!creation.username || !creation.charname || !creation.cls) return alert("Missing fields.");
        const hp = CLASS_HP[creation.cls] || 8;
        const p = { id: Date.now(), ...creation, hp, maxHp: hp, consequences: [], protectionBalance: 0, isDead: false };
        setPlayers(prev => [p, ...prev]);
        logEvent(`Forged new character: ${p.charname}`, "creation");
        setCreation({ username:"", charname:"", cls:"", faction:"Unaligned", status:"Active & Healthy" });
        setTab("players");
      };

      const updatePlayer = (id, updates, logMsg = null) => {
        setPlayers(prev => prev.map(p => p.id === id ? { ...p, ...updates } : p));
        if (logMsg) logEvent(logMsg, "update");
      };

      const adjHP = (id, delta, name) => {
        setPlayers(prev => prev.map(p => {
          if (p.id !== id) return p;
          const newHp = Math.max(0, Math.min(p.maxHp, p.hp + delta));
          logEvent(`Adjusted ${name}'s HP by ${delta > 0 ? '+'+delta : delta} (${newHp}/${p.maxHp})`, delta > 0 ? "heal" : "damage");
          return { ...p, hp: newHp };
        }));
      };

      const executeRoll = () => {
        setDiceAnim(true);
        setTimeout(() => { 
          const roll = d20();
          setDiceRoll(roll); 
          setDiceAnim(false); 
          let tier = roll === 1 ? "FUMBLE" : roll <= 4 ? "FAIL" : roll <= 9 ? "WEAK HIT" : roll <= 14 ? "SUCCESS" : roll <= 19 ? "STRONG HIT" : "CRITICAL";
          logEvent(`Rolled d20: [ ${roll} ] — ${tier}`, "dice");
        }, 400);
      };

      const registerBattle = () => {
        if (!battleP1 || !battleP2 || coins1 === "" || coins2 === "") return alert("Complete all battle fields.");
        const c1 = parseInt(coins1); const c2 = parseInt(coins2);
        const winner = c1 >= c2 ? battleP1 : battleP2;
        const winningCoins = Math.max(c1, c2);
        
        let tier = "⚠ MINOR"; let cons = "Lose title OR stat debuff";
        if (winningCoins >= 45000) { tier = "👁 OP NARRATIVE"; cons = "Winner narrates anything."; }
        else if (winningCoins >= 20000) { tier = "☠ SEVERE"; cons = "Character killed OR curse+exile+theft"; }
        else if (winningCoins >= 10000) { tier = "💀 MAJOR"; cons = "Exile OR servant"; }
        else if (winningCoins >= 5000) { tier = "⚔ MODERATE"; cons = "Cursed 2 sessions OR item stolen"; }

        const battleRecord = { id: Date.now(), date: new Date().toLocaleDateString(), p1: battleP1, c1, p2: battleP2, c2, winner, winningCoins, tier, cons };
        setBattleHistory(prev => [battleRecord, ...prev]);
        logEvent(`Battle Registered: ${winner} wins. Tier: ${tier}`, "battle");
        setBattleP1(""); setBattleP2(""); setCoins1(""); setCoins2("");
        alert("Battle Registered to Ledger.");
      };

      const TABS = [
        { id:"players", label:"👥 Characters" },
        { id:"arena",   label:"⚔️ Arena" },
        { id:"world",   label:"🗺️ Lore" },
        { id:"ledger",  label:"📋 Ledger" },
        { id:"ref",     label:"📖 Rules" },
      ];

      return (
        <div style={S.app} onPointerDown={handleTouch}>
          {ripples.map(r => (
            <div key={r.id} style={{ position: "absolute", left: r.x, top: r.y, width: 100, height: 100, marginLeft: -50, marginTop: -50, background: "radial-gradient(circle, rgba(129, 140, 248, 0.4) 0%, rgba(129, 140, 248, 0) 70%)", borderRadius: "50%", pointerEvents: "none", zIndex: 9999, animation: "rippleAnim 0.8s ease-out forwards" }}/>
          ))}
          <style>{`@keyframes rippleAnim { 0% { transform: scale(0.2); opacity: 1; } 100% { transform: scale(3); opacity: 0; } }`}</style>

          <img src={HERO_IMAGE_URL} alt="Hero" style={S.headerImage} />
          <div style={S.headerGlass}>
            <div style={{ display:"flex", alignItems:"center", justifyContent:"space-between", position: "relative", zIndex: 1 }}>
              <div>
                <div style={{ fontWeight:"800", fontSize:"1.2rem", color:"#fff", letterSpacing:"1px" }}>THE HOLLOW REALM</div>
                <div style={{ fontSize:"0.65rem", color:C.accent, letterSpacing:"2px", textTransform:"uppercase", marginTop:"2px", fontWeight:"600" }}>Game Master Console</div>
              </div>
              <div style={{ background:"rgba(251, 191, 36, 0.15)", border:`1px solid ${C.gold}`, color:C.gold, fontSize:"0.65rem", fontWeight:"700", padding:"4px 12px", borderRadius:"20px", letterSpacing:"1px" }}>LIVE</div>
            </div>
          </div>

          <div style={S.tabs}>
            {TABS.map(t => <div key={t.id} style={S.tab(tab===t.id)} onClick={() => setTab(t.id)}>{t.label}</div>)}
          </div>

          <div style={{ paddingTop: "180px", marginTop: "-180px", pointerEvents:"none" }}></div>

          {tab === "players" && (
            <div style={S.panel}>
              <div style={{ display:"flex", gap:"10px", marginBottom:"16px" }}>
                <input style={{ ...S.input, flex:1 }} placeholder="Search characters..." value={search} onChange={e => setSearch(e.target.value)} />
                <button style={{...S.btn(undefined, undefined, "rgba(255,255,255,0.1)"), width:"auto"}} onClick={() => setTab("create")}>+ Forge</button>
              </div>
              
              {players.length === 0 ? (
                <div style={{ textAlign:"center", padding:"40px 20px", color:C.dim }}>No adventurers logged.</div>
              ) : (
                players.filter(p => p.charname.toLowerCase().includes(search.toLowerCase())).map(p => (
                  <PlayerCard key={p.id} p={p} onAdjHP={adjHP} onSelect={setSelectedPlayer} />
                ))
              )}
            </div>
          )}

          {selectedPlayer && tab === "players" && (() => {
            const p = players.find(x => x.id === selectedPlayer);
            return (
              <div style={{ position:"fixed", inset:0, background:"rgba(0,0,0,0.8)", backdropFilter:"blur(5px)", zIndex:200, padding:"20px", display:"flex", alignItems:"center", justifyContent:"center" }}>
                <div style={{ background:C.bg, border:`1px solid ${C.borderGlow}`, borderRadius:"20px", width:"100%", maxWidth:"450px", padding:"24px" }}>
                  <div style={{ display:"flex", justifyContent:"space-between", marginBottom:"20px" }}>
                    <div><div style={{ fontSize:"1.2rem", fontWeight:"bold" }}>{p.charname}</div><div style={{ fontSize:"0.8rem", color:C.dim }}>@{p.username}</div></div>
                    <button onClick={() => setSelectedPlayer(null)} style={{ background:"none", border:"none", color:C.dim, fontSize:"1.2rem", cursor:"pointer" }}>✕</button>
                  </div>
                  <div style={S.sectionTitle}>Status & Faction</div>
                  <select style={{...S.input, marginBottom:"12px"}} value={p.status} onChange={e => updatePlayer(p.id, { status: e.target.value }, `${p.charname} status changed to ${e.target.value}`)}>
                    {STATUSES.map(s => <option key={s}>{s}</option>)}
                  </select>
                  <select style={{...S.input, marginBottom:"20px"}} value={p.faction} onChange={e => updatePlayer(p.id, { faction: e.target.value }, `${p.charname} aligned with ${e.target.value}`)}>
                    {FACTIONS.map(f => <option key={f}>{f}</option>)}
                  </select>
                  <button style={{...S.btn(), background: p.isDead ? "rgba(251, 191, 36, 0.2)" : "rgba(248, 113, 113, 0.2)", borderColor: p.isDead ? C.gold : C.red, color: p.isDead ? C.gold : C.red }} onClick={() => updatePlayer(p.id, { isDead: !p.isDead, hp: p.isDead ? Math.ceil(p.maxHp/2) : 0 })}>
                    {p.isDead ? "⚗️ Resurrect" : "☠️ Mark Dead"}
                  </button>
                </div>
              </div>
            );
          })()}

          {tab === "create" && (
            <div style={S.panel}>
              <div style={S.card}>
                <div style={S.sectionTitle}>Forge Identity</div>
                <div style={{ marginBottom:"12px" }}><input style={S.input} value={creation.username} onChange={e => setCreation(p=>({...p,username:e.target.value}))} placeholder="@username" /></div>
                <div style={{ marginBottom:"12px" }}>
                  <select style={S.input} value={creation.charname} onChange={e => setCreation(p=>({...p,charname:e.target.value}))}>
                    <option value="">— Select Name —</option>{NAMES.filter(n => !players.find(p => p.charname === n && !p.isDead)).map(n => <option key={n}>{n}</option>)}
                  </select>
                </div>
                <div style={{ marginBottom:"12px" }}>
                  <select style={S.input} value={creation.cls} onChange={e => setCreation(p=>({...p,cls:e.target.value}))}>
                    <option value="">— Select Class —</option>{CLASSES.map(c => <option key={c}>{c}</option>)}
                  </select>
                </div>
                <button style={{ ...S.btn(), marginTop:"10px" }} onClick={addPlayer}>Confirm & Write to Lore</button>
                <button style={{ ...S.btn("transparent", C.dim, "transparent"), marginTop:"5px" }} onClick={()=>setTab("players")}>Cancel</button>
              </div>
            </div>
          )}

          {tab === "arena" && (
            <div style={S.panel}>
              <div style={{...S.card, textAlign:"center", padding:"30px 20px"}}>
                <div style={S.sectionTitle}>Fate Engine</div>
                <div style={{ fontSize:"5rem", filter: diceAnim ? "blur(8px)" : "none", transition:"all 0.3s", transform: diceAnim ? "scale(0.8) rotate(180deg)" : "scale(1) rotate(0deg)" }}>🎲</div>
                <div style={{ height: "60px", marginTop: "10px" }}>
                  {diceRoll && !diceAnim && (
                    <div style={{ animation: "fadeIn 0.4s ease-out" }}>
                      <div style={{ fontSize:"2rem", fontWeight:"bold", color: diceRoll >= 10 ? C.green : C.red }}>{diceRoll}</div>
                    </div>
                  )}
                </div>
                <button style={S.btn("rgba(129, 140, 248, 0.3)")} onClick={executeRoll}>ROLL d20</button>
              </div>

              <div style={S.card}>
                <div style={{...S.sectionTitle, color:C.gold}}>Box Battle Finalization</div>
                <div style={{ display:"flex", gap:"10px", marginBottom:"12px" }}>
                  <div style={{ flex:1 }}><input style={{...S.input, marginBottom:"6px"}} value={battleP1} onChange={e=>setBattleP1(e.target.value)} placeholder="@user1" /><input style={S.input} type="number" value={coins1} onChange={e=>setCoins1(e.target.value)} placeholder="Final Coins" /></div>
                  <div style={{ display:"flex", alignItems:"center", fontWeight:"bold", color:C.accent }}>VS</div>
                  <div style={{ flex:1 }}><input style={{...S.input, marginBottom:"6px"}} value={battleP2} onChange={e=>setBattleP2(e.target.value)} placeholder="@user2" /><input style={S.input} type="number" value={coins2} onChange={e=>setCoins2(e.target.value)} placeholder="Final Coins" /></div>
                </div>
                <button style={S.btn("rgba(251, 191, 36, 0.15)", C.gold, C.gold)} onClick={registerBattle}>Lock In & Register Battle</button>
              </div>
            </div>
          )}

          {tab === "world" && (
            <div style={S.panel}>
              <div style={S.card}>
                <div style={S.sectionTitle}>Narrative State</div>
                <div style={{ marginBottom:"12px" }}><label style={S.label}>Current Arc Title</label><input style={S.input} value={world.arcTitle} onChange={e=>setWorld(p=>({...p, arcTitle:e.target.value}))} /></div>
                <div style={{ marginBottom:"12px" }}><label style={S.label}>Active Villain</label><input style={S.input} value={world.arcVillain} onChange={e=>setWorld(p=>({...p, arcVillain:e.target.value}))} /></div>
                <div style={{ marginBottom:"12px" }}><label style={S.label}>Open Threads</label><textarea style={{...S.input, minHeight:"80px", resize:"vertical"}} value={world.arcThread} onChange={e=>setWorld(p=>({...p, arcThread:e.target.value}))} /></div>
              </div>
            </div>
          )}

          {tab === "ledger" && (
            <div style={S.panel}>
              <div style={S.card}>
                <div style={S.sectionTitle}>Box Battle Records</div>
                {battleHistory.length === 0 ? <div style={{color:C.dim, fontSize:"0.75rem"}}>No battles registered yet.</div> : 
                  battleHistory.map(b => (
                    <div key={b.id} style={{ padding:"10px", background:"rgba(0,0,0,0.3)", borderRadius:"8px", marginBottom:"8px", borderLeft:`2px solid ${C.gold}`}}>
                      <div style={{ fontSize:"0.8rem", fontWeight:"bold", color:"#fff" }}>{b.winner} Defeated {b.winner === b.p1 ? b.p2 : b.p1}</div>
                      <div style={{ fontSize:"0.7rem", color:C.accent, marginTop:"4px" }}>Tier: {b.tier}</div>
                    </div>
                  ))
                }
              </div>
              <div style={S.card}>
                <div style={S.sectionTitle}>Live Action Feed</div>
                <div style={{ maxHeight:"400px", overflowY:"auto" }}>
                  {actionLog.length === 0 ? <div style={{color:C.dim, fontSize:"0.75rem"}}>Feed is clean.</div> :
                    actionLog.map(log => {
                      let color = C.text;
                      if (log.type === "damage") color = C.red;
                      if (log.type === "heal") color = C.green;
                      if (log.type === "dice") color = C.accent;
                      if (log.type === "battle") color = C.gold;
                      return (
                        <div key={log.id} style={{ padding:"8px 0", borderBottom:`1px solid ${C.border}`, display:"flex", gap:"10px", alignItems:"flex-start" }}>
                          <span style={{ fontSize:"0.6rem", color:C.dim, whiteSpace:"nowrap", paddingTop:"3px" }}>{log.time}</span>
                          <span style={{ fontSize:"0.8rem", color:color, lineHeight:"1.4" }}>{log.message}</span>
                        </div>
                      );
                    })
                  }
                </div>
              </div>
            </div>
          )}

          {tab === "ref" && (
            <div style={S.panel}>
              {GIFT_TIERS.map((tierGroup, idx) => (
                <div key={idx} style={S.card}>
                  <div style={S.sectionTitle}>{tierGroup.category}</div>
                  {tierGroup.items.map((item, i) => (
                    <div key={i} style={{ padding:"10px 0", borderBottom: i === tierGroup.items.length - 1 ? "none" : `1px solid ${C.border}` }}>
                      <div style={{ display:"flex", justifyContent:"space-between", alignItems:"baseline" }}>
                        <span style={{ fontWeight:"600", color:"#fff", fontSize:"0.85rem" }}>{item.name}</span>
                        <span style={{ color:C.gold, fontWeight:"bold", fontSize:"0.85rem" }}>{item.coins}</span>
                      </div>
                    </div>
                  ))}
                </div>
              ))}
            </div>
          )}

        </div>
      );
    }

    const root = ReactDOM.createRoot(document.getElementById('root'));
    root.render(<App />);
  </script>
</body>
</html>
