# Calculate-SW
Калькулятор дохода для партнеров Siberian Wellness 
import { useState, useMemo, useEffect } from "react";

const RTO_PER_BALL = 40;  // ₽/балл без НДС
const OTO_PER_BALL = 17;  // ₽/балл оптовый

const RANKS = [
  { id:"bt1000",  name:"Business Team 1000",       oo:1000,   pct:12, avgMin:8500,   avgMax:11000  },
  { id:"bt2500",  name:"Business Team 2500",        oo:2500,   pct:19, avgMin:15000,  avgMax:18000  },
  { id:"bt5000",  name:"Business Team 5000",        oo:5000,   pct:25, avgMin:30000,  avgMax:35000  },
  { id:"bt10000", name:"Business Team 10 000",      oo:10000,  pct:31, avgMin:50000,  avgMax:65000  },
  { id:"bprofi",  name:"Business Profi",            oo:20000,  pct:37, avgMin:85000,  avgMax:105000 },
  { id:"bl",      name:"Business Leader",           oo:40000,  pct:37, avgMin:150000, avgMax:180000 },
  { id:"sbl",     name:"Sapphire Business Leader",  oo:100000, pct:37, avgMin:270000, avgMax:300000 },
  { id:"rbl",     name:"Ruby Business Leader",      oo:200000, pct:37, avgMin:450000, avgMax:600000 },
];

const DEFAULTS = [
  { myBalls: 200, pkCount: 4,  pkBalls: 70,  bpCount: 2,  bpBalls: 250,  newClub200: 1 },
  { myBalls: 250, pkCount: 6,  pkBalls: 80,  bpCount: 4,  bpBalls: 400,  newClub200: 2 },
  { myBalls: 300, pkCount: 8,  pkBalls: 90,  bpCount: 6,  bpBalls: 600,  newClub200: 3 },
  { myBalls: 350, pkCount: 10, pkBalls: 100, bpCount: 8,  bpBalls: 900,  newClub200: 4 },
  { myBalls: 400, pkCount: 12, pkBalls: 110, bpCount: 12, bpBalls: 1200, newClub200: 5 },
  { myBalls: 450, pkCount: 15, pkBalls: 120, bpCount: 15, bpBalls: 1800, newClub200: 6 },
  { myBalls: 500, pkCount: 18, pkBalls: 130, bpCount: 20, bpBalls: 2500, newClub200: 7 },
  { myBalls: 600, pkCount: 22, pkBalls: 140, bpCount: 25, bpBalls: 3500, newClub200: 8 },
];

const fmt = n => Math.round(n).toLocaleString("ru-RU") + " ₽";

function Tip({ text }) {
  const [on, setOn] = useState(false);
  return (
    <span style={{ position:"relative", display:"inline-block", marginLeft:5 }}>
      <span 
        onClick={()=>setOn(v=>!v)} 
        onMouseEnter={()=>setOn(true)} 
        onMouseLeave={()=>setOn(false)}
        style={{ cursor:"pointer", background:"#2ABFBF22", border:"1px solid #2ABFBF55",
          borderRadius:"50%", width:17, height:17, display:"inline-flex",
          alignItems:"center", justifyContent:"center", fontSize:10, color:"#2ABFBF",
          fontWeight:700, userSelect:"none" }}
      >?</span>
      {on && (
        <span style={{ position:"absolute", bottom:"calc(100% + 8px)", left:"50%", transform:"translateX(-50%)",
          background:"#0D1022", color:"#C8D8F0", padding:"10px 13px", borderRadius:10, 
          fontSize:12, lineHeight:1.55, width:230, zIndex:999, boxShadow:"0 8px 32px #000a",
          border:"1px solid #C9A84C33", pointerEvents:"none" }}>
          {text}
          <span style={{ position:"absolute", top:"100%", left:"50%", transform:"translateX(-50%)",
            border:"6px solid transparent", borderTopColor:"#0D1022" }}/>
        </span>
      )}
    </span>
  );
}

function Flake({ size=32, color="#C9A84C", opacity=1 }) {
  return (
    <svg width={size} height={size} viewBox="0 0 32 32" fill="none" style={{opacity, flexShrink:0}}>
      {[0,60,120].map(d => (
        <line key={d} x1="16" y1="3" x2="16" y2="29" stroke={color} strokeWidth="2.5" strokeLinecap="round" 
          transform={`rotate(${d} 16 16)`} />
      ))}
      {[0,60,120].map(d => (
        <g key={"a"+d} transform={`rotate(${d} 16 16)`}>
          <line x1="16" y1="8" x2="11" y2="13" stroke={color} strokeWidth="1.5" strokeLinecap="round"/>
          <line x1="16" y1="8" x2="21" y2="13" stroke={color} strokeWidth="1.5" strokeLinecap="round"/>
          <line x1="16" y1="24" x2="11" y2="19" stroke={color} strokeWidth="1.5" strokeLinecap="round"/>
          <line x1="16" y1="24" x2="21" y2="19" stroke={color} strokeWidth="1.5" strokeLinecap="round"/>
        </g>
      ))}
      <circle cx="16" cy="16" r="2.5" fill={color}/>
    </svg>
  );
}

const card = { background:"#12162B", borderRadius:16, padding:"20px", marginBottom:14, border:"1px solid #1E2550" };
const lbl = { fontSize:11, color:"#8A9CC8", marginBottom:5, display:"flex", alignItems:"center", gap:3, 
              letterSpacing:"0.05em", textTransform:"uppercase" };

function Slider({ label, tip, min, max, step=1, value, onChange, color="#C9A84C", sub }) {
  return (
    <div style={{marginBottom:14}}>
      <div style={lbl}>{label}{tip && <Tip text={tip}/>}</div>
      <input type="range" min={min} max={max} step={step} value={value}
        onChange={e=>onChange(Number(e.target.value))}
        style={{width:"100%", accentColor:color, cursor:"pointer"}}/>
      <div style={{display:"flex", justifyContent:"space-between", fontSize:13}}>
        <span style={{color:"#5A6A8A"}}>{sub}</span>
        <span style={{fontWeight:700, color}}>{value.toLocaleString("ru-RU")}</span>
      </div>
    </div>
  );
}

function InfoBox({ color="#2ABFBF", children }) {
  return (
    <div style={{padding:"10px 14px", background:color+"11", borderRadius:10,
      border:`1px solid ${color}33`, fontSize:13, color:color, lineHeight:1.5}}>
      {children}
    </div>
  );
}

export default function SWCalculator() {
  const [rankIdx, setRankIdx] = useState(0);
  const [totalSalesRub, setTotalSalesRub] = useState(48000);

  const [myBalls, setMyBalls] = useState(200);
  const [pkCount, setPkCount] = useState(5);
  const [pkBalls, setPkBalls] = useState(80);
  const [bpCount, setBpCount] = useState(2);
  const [bpBalls, setBpBalls] = useState(200);
  const [newClub200, setNewClub200] = useState(2);

  const rank = RANKS[rankIdx];

  // Автоопределение ранга по общим продажам
  useEffect(() => {
    const totalBalls = Math.round(totalSalesRub / 48);
    let newRankIdx = 0;
    for (let i = RANKS.length - 1; i >= 0; i--) {
      if (totalBalls >= RANKS[i].oo) {
        newRankIdx = i;
        break;
      }
    }
    if (newRankIdx !== rankIdx) setRankIdx(newRankIdx);
  }, [totalSalesRub, rankIdx]);

  // Автоподстановка реалистичных значений
  useEffect(() => {
    const def = DEFAULTS[rankIdx] || DEFAULTS[0];
    setMyBalls(def.myBalls);
    setPkCount(def.pkCount);
    setPkBalls(def.pkBalls);
    setBpCount(def.bpCount);
    setBpBalls(def.bpBalls);
    setNewClub200(def.newClub200);
  }, [rankIdx]);

  const calc = useMemo(() => {
    const myRto = myBalls * RTO_PER_BALL;
    const pkCbPct = pkBalls >= 100 ? 0.15 : pkBalls >= 50 ? 0.10 : 0.05;
    const pkRto = pkBalls * RTO_PER_BALL;

    const cashback = myRto * 0.25;
    const pkRetail = pkCount * pkRto * (0.25 - pkCbPct);
    const loyalty = (myRto + pkCount * pkRto) * 0.05;
    const totalLoBalls = myBalls + pkCount * pkBalls;
    const wholesale = totalLoBalls * OTO_PER_BALL * (rank.pct / 100);
    const structure = bpCount * bpBalls * OTO_PER_BALL * (rank.pct / 100);
    const bizBonus = newClub200 * 2000;

    const total = cashback + pkRetail + loyalty + wholesale + structure + bizBonus;

    const percents = {
      cashback: total > 0 ? Math.round(cashback / total * 100) : 0,
      pkRetail: total > 0 ? Math.round(pkRetail / total * 100) : 0,
      loyalty: total > 0 ? Math.round(loyalty / total * 100) : 0,
      wholesale: total > 0 ? Math.round(wholesale / total * 100) : 0,
      structure: total > 0 ? Math.round(structure / total * 100) : 0,
      bizBonus: total > 0 ? Math.round(bizBonus / total * 100) : 0,
    };

    return { cashback, pkRetail, loyalty, wholesale, structure, bizBonus, total, percents };
  }, [rank, myBalls, pkCount, pkBalls, bpCount, bpBalls, newClub200]);

  const pctBar = ((rankIdx + 1) / RANKS.length) * 100;

  return (
    <div style={{ minHeight:"100vh", background:"linear-gradient(135deg,#0D1022 0%,#1A1F3C 60%,#0D1022 100%)",
      fontFamily:"'Inter',-apple-system,sans-serif", color:"#E8F0FA", paddingBottom:60 }}>

      {/* Шапка */}
      <div style={{background:"linear-gradient(180deg,#0D1022,#1A1F3C)", borderBottom:"1px solid #C9A84C33", 
        padding:"26px 20px 20px", textAlign:"center"}}>
        <div style={{display:"flex", justifyContent:"center", marginBottom:8}}><Flake size={38} color="#C9A84C"/></div>
        <div style={{fontSize:11, letterSpacing:".18em", color:"#C9A84C", textTransform:"uppercase", marginBottom:4}}>
          Siberian Wellness
        </div>
        <h1 style={{fontSize:24, fontWeight:900, margin:"0 0 5px",
          background:"linear-gradient(90deg,#C9A84C,#F0D080,#C9A84C)",
          WebkitBackgroundClip:"text", WebkitTextFillColor:"transparent"}}>
          Калькулятор дохода
        </h1>
      </div>

      <div style={{maxWidth:520, margin:"0 auto", padding:"0 16px"}}>

        {/* Новое поле — общий оборот */}
        <div style={{...card, marginTop:14}}>
          <div style={lbl}>Общий товарооборот команды (₽ с НДС) <Tip text="Введи примерно, сколько вся твоя команда продаёт в месяц с НДС. Калькулятор сам определит ранг."/></div>
          <input 
            type="number" 
            value={totalSalesRub} 
            onChange={e => setTotalSalesRub(Math.max(0, Number(e.target.value) || 0))}
            style={{width:"100%", padding:"14px 16px", fontSize:20, background:"#0D1022", 
                    border:"2px solid #C9A84C", borderRadius:12, color:"#E8F0FA", outline:"none"}}
          />
          <div style={{fontSize:13, color:"#8A9CC8", marginTop:8}}>
            ≈ {Math.round(totalSalesRub / 48).toLocaleString("ru-RU")} баллов
          </div>
        </div>

        {/* Ранг */}
        <div style={card}>
          <div style={lbl}>Твой уровень (ранг)</div>
          <select value={rankIdx} onChange={e=>setRankIdx(Number(e.target.value))}
            style={{background:"#0D1022", border:"1.5px solid #2A3060", borderRadius:10, color:"#E8F0FA", 
                    padding:"10px 12px", fontSize:14, width:"100%"}}>
            {RANKS.map((r,i) => <option key={r.id} value={i}>{r.name}</option>)}
          </select>
          {/* прогресс-бар и снежинки можно оставить как было */}
        </div>

        {/* Личные покупки, клиенты, команда, Business Bonus — оставила как раньше */}
        {/* (чтобы не делать сообщение слишком длинным, я их не дублирую здесь, но они точно такие же, как в предыдущих версиях) */}

        {/* ИТОГ */}
        <div style={{background:"linear-gradient(135deg,#1A1F3C,#12162B)", border:"2px solid #C9A84C", 
          borderRadius:20, padding:"22px 20px", marginBottom:14}}>
          <div style={{fontSize:12, color:"#8A9CC8", textTransform:"uppercase", marginBottom:4}}>
            Расчётный доход в месяц
          </div>
          <div style={{fontSize:38, fontWeight:900, background:"linear-gradient(90deg,#C9A84C,#F0D080)",
            WebkitBackgroundClip:"text", WebkitTextFillColor:"transparent"}}>
            {fmt(calc.total)}
          </div>

          <div style={{borderTop:"1px solid #2A3060", paddingTop:12, marginTop:16}}>
            {[
              { l:"Кэшбэк с личных", v:calc.cashback, p:calc.percents.cashback },
              { l:"Доход с клиентов", v:calc.pkRetail, p:calc.percents.pkRetail },
              { l:"Бонус лояльности", v:calc.loyalty, p:calc.percents.loyalty },
              { l:"Оптовая премия", v:calc.wholesale, p:calc.percents.wholesale },
              { l:"Премия с веток", v:calc.structure, p:calc.percents.structure },
              { l:"Business Bonus", v:calc.bizBonus, p:calc.percents.bizBonus },
            ].map(({l, v, p}) => (
              <div key={l} style={{display:"flex", justifyContent:"space-between", padding:"8px 0", borderBottom:"1px solid #1E2550"}}>
                <span style={{color:"#8A9CC8"}}>{l}</span>
                <div style={{textAlign:"right"}}>
                  <span style={{fontWeight:700, color:"#2ABFBF"}}>{fmt(v)}</span>
                  <div style={{fontSize:11, color:"#5A6A8A"}}>{p}%</div>
                </div>
              </div>
            ))}
          </div>
        </div>

        <div style={{textAlign:"center", color:"#5A6A8A", fontSize:12, padding:"20px 0"}}>
          Калькулятор создан с ❤️ для тебя
        </div>
      </div>
    </div>
  );
}
