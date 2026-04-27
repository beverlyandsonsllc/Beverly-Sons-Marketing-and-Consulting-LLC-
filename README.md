import { useState, useEffect, useRef } from "react";

// ── IP LIBRARY (from Python) ────────────────────────────────────────────
const IP_LIBRARY = {
  oppress:  { title:"OPPRESS: The American Cage",      focus:"systemic awareness, extraction, freedom",      color:"#E05C5C", icon:"⚖",  emoji:"🏛" },
  teaching: { title:"Legacy Code: T.E.A.C.H.I.N.G.",  focus:"education, empowerment, growth",               color:"#7BC97B", icon:"📚", emoji:"🔥" },
  mirror:   { title:"God in the Mirror",               focus:"self-awareness, household order, identity",    color:"#C9A84C", icon:"🪞", emoji:"☀" },
  wealth:   { title:"Family Wealth Structure",         focus:"money, legacy, governance",                    color:"#B07BD4", icon:"💰", emoji:"🏦" },
  hhos:     { title:"H.H.O.S.",                        focus:"mental health, peace, emotional regulation",   color:"#00E5C8", icon:"🕊", emoji:"⚡" },
  trinity:  { title:"Trinity Path of Ma'at",           focus:"alignment, knowledge, culture, legacy",        color:"#D4846A", icon:"△",  emoji:"🌱" },
  key114:   { title:"Key of 114 — Supreme Math",       focus:"divine order, structure, equality",            color:"#7B9ED9", icon:"🔢", emoji:"👁" },
};

// ── PERSONAS (from Python BeverlyCloneEngine) ───────────────────────────
const PERSONAS = {
  DEQUAN: { label:"Dequan",  desc:"Direct, strategic, legacy-focused",               color:"#C9A84C" },
  WEALTH: { label:"Wealth",  desc:"Financial architect, concise, practical",          color:"#7BC97B" },
  TEACH:  { label:"Teach",   desc:"Mentor, educator, lesson-based",                   color:"#7B9ED9" },
  MIRROR: { label:"Mirror",  desc:"Reflective, sovereign, grounding",                 color:"#B07BD4" },
  HHOS:   { label:"H.H.O.S.",desc:"Calm, healing, emotionally intelligent",           color:"#00E5C8" },
};

const PLATFORMS = ["Instagram","TikTok","Twitter/X","LinkedIn","Facebook","WhatsApp"];

const SYSTEM_PROMPT = `You are the Beverly & Sons Daily IP Post Engine — BeverlyCloneEngine.

Generate a complete daily content post for the given IP book and persona.

RETURN ONLY raw JSON:
{
  "title": "IP book title",
  "hook": "Attention-grabbing first line (1 sentence, BOLD energy)",
  "lesson": "2-3 sentence teaching rooted in the IP focus. Apply the persona lens.",
  "affirmation": "First-person affirmation (1 sentence, present tense)",
  "poem": "2-4 line cipher/spoken word closing (rhyme optional but tight)",
  "cta": "Platform-specific call to action",
  "hashtags": ["array", "of", "5", "relevant", "hashtags"],
  "platform_versions": {
    "instagram": "Full post optimized for Instagram (hook + lesson + affirmation + hashtags)",
    "tiktok": "[HOOK 0-3s] text\n[VALUE 3-45s] text\n[CTA 45-60s] text",
    "twitter": "Under 280 chars — hardest hitting version",
    "linkedin": "Professional framing, 100-150 words, story → framework → lesson",
    "whatsapp": "Short, personal, conversational. 2-3 sentences."
  },
  "calendar_description": "Full post content formatted for Google Calendar event description",
  "supreme_math_connection": "How this IP connects to Supreme Mathematics"
}

VOICE — ${"{"}persona{"}"}:
Always direct. Always earned. Never preachy. Beverly & Sons cadence.`;

const DAYS_OF_WEEK = ["Sun","Mon","Tue","Wed","Thu","Fri","Sat"];
const HOURS = Array.from({length:24},(_,i)=>i);

const G="#C9A84C", N="#050a12", NL="#0d1520", C="#e8d9b0";

const css=`
@import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;600;700;900&family=Rajdhani:wght@400;500;600;700&family=Cormorant+Garamond:ital,wght@0,400;0,600;0,700;1,400&family=DM+Sans:wght@300;400;500;600&display=swap');
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0;}
.root{font-family:'Rajdhani',sans-serif;background:${N};min-height:100vh;color:${C};overflow-x:hidden;}
.bg{position:fixed;inset:0;z-index:0;pointer-events:none;
  background:radial-gradient(ellipse 60% 50% at 50% 0%,#1a2a0a,transparent 60%),
  radial-gradient(ellipse 40% 40% at 10% 90%,#001020,transparent 50%),${N};}
.grid{position:fixed;inset:0;z-index:0;pointer-events:none;
  background-image:linear-gradient(rgba(0,229,200,0.015) 1px,transparent 1px),linear-gradient(90deg,rgba(201,168,76,0.01) 1px,transparent 1px);
  background-size:40px 40px;}
@keyframes sc{0%{top:-2px}100%{top:100%}}
.scanline{position:fixed;left:0;right:0;height:1px;z-index:200;pointer-events:none;background:rgba(0,229,200,0.04);animation:sc 12s linear infinite;}

.layout{position:relative;z-index:1;display:grid;grid-template-columns:260px 1fr;height:100vh;max-width:1300px;margin:0 auto;}
@media(max-width:700px){.layout{grid-template-columns:1fr;height:auto;}}

/* SIDEBAR */
.side{border-right:1px solid rgba(0,229,200,0.08);display:flex;flex-direction:column;overflow:hidden;}
.side-hdr{padding:18px 16px 14px;border-bottom:1px solid rgba(0,229,200,0.08);}
.s-logo{display:flex;align-items:center;gap:10px;}
.s-badge{width:38px;height:38px;border-radius:50%;border:1.5px solid ${G};background:rgba(201,168,76,0.07);display:flex;align-items:center;justify-content:center;font-family:'Orbitron',monospace;font-size:9px;font-weight:700;color:${G};flex-shrink:0;}
.s-name{font-family:'Orbitron',monospace;font-size:13px;font-weight:700;color:${C};letter-spacing:.5px;}
.s-name span{color:${G};}
.s-sub{font-size:8px;letter-spacing:2.5px;text-transform:uppercase;color:rgba(0,229,200,0.35);margin-top:2px;}

/* IP SELECTOR */
.ip-section{padding:10px 10px 0;}
.sec-label{font-size:8px;letter-spacing:2.5px;text-transform:uppercase;color:rgba(232,217,176,0.25);padding:0 6px;margin-bottom:6px;font-family:'Orbitron',monospace;}
.ip-list{display:flex;flex-direction:column;gap:3px;}
.ip-item{padding:9px 10px;border-radius:4px;border:1px solid transparent;cursor:pointer;transition:all .15s;display:flex;align-items:center;gap:8px;}
.ip-item:hover,.ip-item.a{background:rgba(0,0,0,0.3);border-color:var(--ic);}
.ip-icon{font-size:15px;flex-shrink:0;}
.ip-name{font-size:11px;font-weight:600;color:rgba(232,217,176,0.65);line-height:1.2;}
.ip-item.a .ip-name{color:var(--ic);}
.ip-focus{font-size:9px;color:rgba(232,217,176,0.25);margin-top:1px;}

/* PERSONA */
.persona-grid{display:grid;grid-template-columns:1fr 1fr;gap:5px;padding:0 10px;}
.persona-btn{padding:7px 6px;border-radius:4px;border:1px solid rgba(255,255,255,0.07);background:rgba(255,255,255,0.02);cursor:pointer;transition:all .15s;text-align:center;}
.persona-btn:hover,.persona-btn.a{border-color:var(--pc);background:rgba(0,0,0,0.3);}
.pb-name{font-size:10px;font-weight:700;font-family:'Orbitron',monospace;color:var(--pc);display:block;}
.pb-desc{font-size:8px;color:rgba(232,217,176,0.3);display:block;margin-top:1px;}

/* SCHEDULE CONFIG */
.sched-box{padding:10px;}
.s-row{display:grid;grid-template-columns:1fr 1fr;gap:6px;margin-bottom:8px;}
.s-field{display:flex;flex-direction:column;gap:3px;}
.s-lbl{font-size:8px;letter-spacing:2px;text-transform:uppercase;color:rgba(0,229,200,0.4);font-family:'Orbitron',monospace;}
.s-select{background:rgba(0,229,200,0.03);border:1px solid rgba(0,229,200,0.12);border-radius:3px;padding:6px 8px;color:${C};font-family:'Rajdhani',sans-serif;font-size:12px;outline:none;}
.s-select option{background:#0d1520;}
.plat-chips{display:flex;flex-wrap:wrap;gap:4px;}
.plat-chip{padding:3px 7px;border-radius:10px;border:1px solid rgba(255,255,255,0.08);background:rgba(255,255,255,0.03);cursor:pointer;font-size:10px;font-weight:600;color:rgba(232,217,176,0.4);transition:all .15s;}
.plat-chip.a{border-color:${G};background:rgba(201,168,76,0.1);color:${G};}

/* GEN BTN */
.gen-btn{width:100%;padding:12px;background:${G};border:none;border-radius:4px;color:#050a12;font-family:'Orbitron',monospace;font-size:11px;font-weight:700;letter-spacing:1px;cursor:pointer;transition:all .15s;margin-top:4px;}
.gen-btn:hover:not(:disabled){background:#d4a832;box-shadow:0 0 20px rgba(201,168,76,.35);}
.gen-btn:disabled{opacity:.35;cursor:not-allowed;}
.cal-btn{width:100%;padding:10px;background:rgba(99,102,241,0.12);border:1px solid rgba(99,102,241,0.3);border-radius:4px;color:rgba(99,102,241,0.8);font-family:'Orbitron',monospace;font-size:10px;font-weight:700;letter-spacing:1px;cursor:pointer;transition:all .15s;margin-top:6px;}
.cal-btn:hover:not(:disabled){background:rgba(99,102,241,0.2);box-shadow:0 0 14px rgba(99,102,241,.2);}
.cal-btn:disabled{opacity:.35;cursor:not-allowed;}

/* MAIN CONTENT */
.main{display:flex;flex-direction:column;overflow:hidden;}
.topbar{padding:14px 20px;border-bottom:1px solid rgba(0,229,200,0.08);display:flex;align-items:center;justify-content:space-between;flex-shrink:0;background:rgba(0,0,0,0.15);}
.tb-title{font-family:'Orbitron',monospace;font-size:12px;font-weight:700;color:${C};letter-spacing:1px;}
.tb-right{display:flex;gap:6px;}
.tb-tag{padding:4px 10px;border:1px solid;border-radius:2px;font-size:8px;letter-spacing:1.5px;text-transform:uppercase;font-family:'Orbitron',monospace;}
.tg{color:#7BC97B;border-color:rgba(123,201,123,0.3);background:rgba(123,201,123,0.05);}
.tc{color:#00E5C8;border-color:rgba(0,229,200,0.3);background:rgba(0,229,200,0.05);}

.content{flex:1;overflow-y:auto;padding:18px 20px 60px;}
.content::-webkit-scrollbar{width:2px;}
.content::-webkit-scrollbar-thumb{background:rgba(201,168,76,0.15);}

/* TABS */
.tabs{display:flex;gap:2px;border-bottom:1px solid rgba(255,255,255,0.05);margin-bottom:16px;}
.tab{padding:8px 14px;font-family:'Orbitron',monospace;font-size:9px;letter-spacing:1.5px;color:rgba(232,217,176,0.3);cursor:pointer;border-bottom:2px solid transparent;transition:all .15s;white-space:nowrap;}
.tab:hover,.tab.a{color:${G};border-bottom-color:${G};}

/* POST DISPLAY */
.post-card{background:${NL};border:1px solid rgba(255,255,255,0.06);border-radius:6px;overflow:hidden;margin-bottom:14px;}
.pc-hdr{padding:12px 16px;border-bottom:1px solid rgba(255,255,255,0.05);background:rgba(0,0,0,0.25);display:flex;align-items:center;justify-content:space-between;}
.pc-title{font-family:'Orbitron',monospace;font-size:10px;font-weight:700;color:var(--cc,${G});letter-spacing:1.5px;}
.copy-btn{padding:4px 10px;background:transparent;border:1px solid rgba(255,255,255,0.1);border-radius:2px;color:rgba(232,217,176,0.4);font-size:9px;font-weight:600;letter-spacing:1px;cursor:pointer;transition:all .15s;font-family:'Orbitron',monospace;}
.copy-btn:hover{border-color:var(--cc,${G});color:var(--cc,${G});}
.pc-body{padding:14px 16px;font-family:'Cormorant Garamond',serif;font-size:17px;color:rgba(232,217,176,0.88);line-height:1.8;white-space:pre-wrap;}
.pc-body.small{font-size:14px;font-family:'DM Sans',sans-serif;}
.affirmation-seal{padding:12px 16px;border-left:3px solid var(--cc,${G});background:rgba(0,0,0,0.2);font-family:'Cormorant Garamond',serif;font-style:italic;font-size:16px;color:var(--cc,${G});line-height:1.5;}

/* PLATFORM TABS */
.plat-tabs{display:flex;gap:4px;flex-wrap:wrap;margin-bottom:12px;}
.plat-tab{padding:5px 10px;border-radius:12px;border:1px solid rgba(255,255,255,0.08);background:rgba(255,255,255,0.03);color:rgba(232,217,176,0.4);font-size:10px;font-weight:600;cursor:pointer;transition:all .15s;}
.plat-tab:hover,.plat-tab.a{border-color:${G};background:rgba(201,168,76,0.1);color:${G};}

/* CALENDAR VIEW */
.cal-grid{display:grid;grid-template-columns:repeat(7,1fr);gap:4px;margin-bottom:16px;}
.cal-day{text-align:center;padding:4px;font-size:9px;letter-spacing:1px;text-transform:uppercase;color:rgba(232,217,176,0.3);font-family:'Orbitron',monospace;}
.cal-cell{background:${NL};border:1px solid rgba(255,255,255,0.05);border-radius:4px;padding:8px 6px;text-align:center;min-height:70px;cursor:pointer;transition:all .15s;position:relative;}
.cal-cell:hover{border-color:rgba(255,255,255,0.12);}
.cal-cell.has-event{border-color:var(--ec);background:rgba(0,0,0,0.3);}
.cc-num{font-family:'Orbitron',monospace;font-size:12px;font-weight:700;color:rgba(232,217,176,0.5);margin-bottom:4px;}
.cal-cell.has-event .cc-num{color:${C};}
.cc-event{font-size:8px;padding:2px 4px;border-radius:2px;border:1px solid var(--ec);color:var(--ec);background:rgba(0,0,0,0.3);line-height:1.3;margin-bottom:2px;text-align:left;}

/* JOB LOG */
.job-log{display:flex;flex-direction:column;gap:6px;}
.job-row{padding:11px 14px;background:${NL};border:1px solid rgba(255,255,255,0.05);border-radius:4px;display:flex;align-items:center;justify-content:space-between;cursor:pointer;transition:all .15s;}
.job-row:hover{border-color:rgba(255,255,255,0.1);}
.jr-ip{font-size:10px;font-weight:600;color:var(--jc);font-family:'Orbitron',monospace;letter-spacing:.5px;}
.jr-meta{font-size:11px;color:rgba(232,217,176,0.45);margin-top:2px;}
.jr-right{text-align:right;}
.jr-persona{font-size:9px;letter-spacing:1.5px;font-family:'Orbitron',monospace;color:rgba(232,217,176,0.3);}
.jr-time{font-size:9px;color:rgba(232,217,176,0.2);margin-top:2px;}
.jr-status{padding:2px 7px;border-radius:2px;font-size:8px;font-family:'Orbitron',monospace;}
.st-sched{background:rgba(123,201,123,0.1);border:1px solid rgba(123,201,123,0.3);color:#7BC97B;}
.st-gen{background:rgba(201,168,76,0.1);border:1px solid rgba(201,168,76,0.2);color:${G};}

@keyframes spin{from{transform:rotate(0)}to{transform:rotate(360deg)}}
.spin-ring{width:52px;height:52px;border-radius:50%;border:2px solid rgba(201,168,76,0.1);border-top-color:rgba(201,168,76,0.6);border-right-color:rgba(0,229,200,0.4);animation:spin 1s linear infinite;margin:0 auto 14px;}
.loading-wrap{text-align:center;padding:44px;}
.loading-txt{font-family:'Orbitron',monospace;font-size:10px;color:rgba(201,168,76,0.5);letter-spacing:3px;}
.empty{text-align:center;padding:40px;font-family:'Cormorant Garamond',serif;font-style:italic;font-size:18px;color:rgba(232,217,176,0.2);}
.err{background:rgba(180,40,40,.1);border:1px solid rgba(180,40,40,.3);border-radius:4px;padding:12px;color:#f99;font-size:12px;margin-bottom:12px;}
.hashtags{display:flex;flex-wrap:wrap;gap:5px;padding:12px 16px;border-top:1px solid rgba(255,255,255,0.04);}
.ht{padding:3px 8px;border:1px solid rgba(201,168,76,0.2);border-radius:10px;font-size:10px;color:rgba(201,168,76,0.6);}
.sm-connection{padding:10px 14px;background:rgba(0,229,200,0.04);border:1px solid rgba(0,229,200,0.1);border-radius:4px;font-size:12px;color:rgba(0,229,200,0.6);margin-bottom:14px;}
.sm-lbl{font-size:8px;letter-spacing:2px;text-transform:uppercase;color:rgba(0,229,200,0.4);font-family:'Orbitron',monospace;margin-bottom:4px;}
`;

function getCalDays() {
  const now = new Date();
  const year = now.getFullYear(), month = now.getMonth();
  const first = new Date(year, month, 1).getDay();
  const last = new Date(year, month+1, 0).getDate();
  return {year, month, first, last, today: now.getDate()};
}

function fmtTime(){return new Date().toLocaleTimeString("en-US",{hour:"2-digit",minute:"2-digit"});}
function fmtDate(){return new Date().toLocaleDateString("en-US",{weekday:"short",month:"short",day:"numeric"});}
function hrLabel(h){return `${h===0?"12":h>12?h-12:h}:00 ${h<12?"AM":"PM"}`;}

export default function App() {
  const [selIP, setSelIP] = useState("oppress");
  const [selPersona, setSelPersona] = useState("DEQUAN");
  const [selHour, setSelHour] = useState(8);
  const [selPlatforms, setSelPlatforms] = useState(["Instagram","TikTok"]);
  const [recurring, setRecurring] = useState(true);
  const [tab, setTab] = useState("post");
  const [platTab, setPlatTab] = useState("instagram");
  const [post, setPost] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [jobs, setJobs] = useState([
    {ip:"teaching",persona:"TEACH",platform:["Instagram"],hour:8,status:"scheduled",time:"08:00 AM",date:fmtDate(),hook:"[Legacy Code] Today's truth: education, empowerment, growth."},
    {ip:"hhos",    persona:"HHOS", platform:["TikTok"],   hour:12,status:"scheduled",time:"12:00 PM",date:fmtDate(),hook:"[H.H.O.S.] Today's truth: mental health, peace, emotional regulation."},
  ]);
  const [calEvents, setCalEvents] = useState({3:["hhos"],7:["teaching"],12:["oppress"],15:["wealth"],20:["mirror"],26:["teaching","hhos"]});
  const [copied, setCopied] = useState("");

  const ip = IP_LIBRARY[selIP];
  const persona = PERSONAS[selPersona];
  const cal = getCalDays();

  function togglePlat(p){
    setSelPlatforms(sp => sp.includes(p) ? sp.filter(x=>x!==p) : [...sp,p]);
  }

  async function generatePost() {
    setLoading(true); setError(null); setPost(null);
    try {
      const res = await fetch("https://api.anthropic.com/v1/messages",{
        method:"POST",headers:{"Content-Type":"application/json"},
        body:JSON.stringify({
          model:"claude-sonnet-4-20250514",max_tokens:2000,
          system:SYSTEM_PROMPT.replace("{persona}",`${selPersona} — ${persona.desc}`),
          messages:[{role:"user",content:
            `Generate daily IP post:\nIP Book: ${ip.title}\nFocus: ${ip.focus}\nPersona: ${selPersona} (${persona.desc})\nPlatforms: ${selPlatforms.join(", ")}\nScheduled: ${hrLabel(selHour)} · Recurring: ${recurring}\n\nIMPORTANT: Return ONLY valid raw JSON. Keep all string values concise (under 300 chars each). No markdown.`
          }]
        })
      });
      const data = await res.json();
      if(data.error) throw new Error(data.error.message);
      const raw = data.content?.find(b=>b.type==="text")?.text||"";
      const clean = raw.replace(/```json|```/g,"").trim();

      let parsed;
      try {
        parsed = JSON.parse(clean);
      } catch {
        // Attempt to repair truncated JSON by closing open structures
        let repaired = clean;
        const openBraces = (repaired.match(/{/g)||[]).length - (repaired.match(/}/g)||[]).length;
        const openBrackets = (repaired.match(/\[/g)||[]).length - (repaired.match(/]/g)||[]).length;
        // Close any unterminated string
        if((repaired.match(/"/g)||[]).length % 2 !== 0) repaired += '"';
        // Close arrays and objects
        for(let i=0;i<openBrackets;i++) repaired += "]";
        for(let i=0;i<openBraces;i++) repaired += "}";
        try {
          parsed = JSON.parse(repaired);
        } catch {
          // Last resort: extract known fields manually
          const get = (key) => { const m = clean.match(new RegExp(`"${key}"\\s*:\\s*"([^"]{0,500})"`)); return m?m[1]:""; };
          parsed = {
            title: get("title") || ip.title,
            hook: get("hook") || `[${ip.title}] Today's truth: ${ip.focus}.`,
            lesson: get("lesson") || `Lesson: ${ip.focus} is not just an idea — it is a practice.`,
            affirmation: get("affirmation") || "I build with discipline, clarity, and purpose.",
            poem: get("poem") || "What you repeat, you create. What you protect, you grow.",
            cta: get("cta") || "Scan. Read. Learn. Apply.",
            hashtags: ["BeverlySons","Legacy","HHOS","Empowerment","CAAVE"],
            platform_versions: { instagram: get("instagram") || get("hook"), tiktok:"", twitter:"", linkedin:"", whatsapp:"" },
            calendar_description: get("calendar_description") || get("lesson"),
            supreme_math_connection: get("supreme_math_connection") || ""
          };
        }
      }
      setPost(parsed);
      // add to jobs
      const newJob = {ip:selIP,persona:selPersona,platform:selPlatforms,hour:selHour,status:"generated",time:fmtTime(),date:fmtDate(),hook:parsed.hook};
      setJobs(j=>[newJob,...j]);
      setTab("post");
    } catch(e){setError(e.message);}
    finally{setLoading(false);}
  }

  function scheduleToCalendar() {
    if(!post) return;
    const day = new Date().getDate();
    setCalEvents(ce=>({...ce,[day]:[...(ce[day]||[]),selIP]}));
    const job = jobs.find(j=>j.hook===post?.hook);
    if(job) setJobs(j=>j.map(jj=>jj.hook===job.hook?{...jj,status:"scheduled"}:jj));
    alert(`✅ Event scheduled: "${ip.title}" at ${hrLabel(selHour)}\nRecurring: ${recurring?"Daily":"One-time"}\n\nIn production this calls:\nGET /schedule/${selIP}?tone=${selPersona}&hour=${s