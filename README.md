# App
A responsive Construction Field Management app built with React.js &amp; Vite. Features mock authentication, project dashboard with search/filter, and a Daily Progress Report (DPR) form with photo upload and validation. Includes dark/light mode toggle 
```jsx
import { useState, useEffect, useRef, useCallback } from "react";

// ============================================================
// CONSTANTS
// ============================================================
const PROJECTS = [
  {
    id: "PRJ-001",
    name: "Horizon Tower Complex",
    status: "active",
    startDate: "2025-03-01",
    endDate: "2026-08-31",
    location: "Mumbai, Maharashtra",
    client: "Horizon Realty Ltd.",
    progress: 67,
    manager: "Alex Morgan",
    budget: "₹48.5 Cr",
    description: "42-storey mixed-use tower with 280 residential units and commercial podium.",
  },
  {
    id: "PRJ-002",
    name: "NH-48 Highway Expansion",
    status: "active",
    startDate: "2025-06-15",
    endDate: "2027-02-28",
    location: "Pune–Mumbai Corridor",
    client: "NHAI / PWD Maharashtra",
    progress: 31,
    manager: "Priya Sharma",
    budget: "₹210 Cr",
    description: "68 km six-lane highway expansion with 3 flyovers and 2 underpasses.",
  },
  {
    id: "PRJ-003",
    name: "Greenfield IT Campus",
    status: "pending",
    startDate: "2026-01-10",
    endDate: "2027-06-30",
    location: "Hyderabad, Telangana",
    client: "TechPark Developers Pvt. Ltd.",
    progress: 8,
    manager: "Rohan Mehta",
    budget: "₹95 Cr",
    description: "Phased IT campus development — 3 blocks, data centre, and central amenities.",
  },
  {
    id: "PRJ-004",
    name: "Coastal Bridge Rehabilitation",
    status: "onhold",
    startDate: "2024-11-01",
    endDate: "2025-12-31",
    location: "Kochi, Kerala",
    client: "Kerala PWD",
    progress: 44,
    manager: "Sunil Nair",
    budget: "₹32 Cr",
    description: "Structural rehabilitation of 1.2 km coastal bridge with seismic retrofitting.",
  },
  {
    id: "PRJ-005",
    name: "Smart Water Treatment Plant",
    status: "completed",
    startDate: "2024-02-01",
    endDate: "2025-10-31",
    location: "Bengaluru, Karnataka",
    client: "BWSSB / Smart City Mission",
    progress: 100,
    manager: "Ananya Iyer",
    budget: "₹67 Cr",
    description: "120 MLD capacity water treatment plant with IoT monitoring systems.",
  },
];

const WEATHER_OPTIONS = [
  { value: "", label: "Select weather conditions" },
  { value: "sunny", label: "☀️  Sunny & Clear" },
  { value: "partly_cloudy", label: "⛅  Partly Cloudy" },
  { value: "cloudy", label: "☁️  Overcast / Cloudy" },
  { value: "rainy", label: "🌧️  Rainy" },
  { value: "heavy_rain", label: "⛈️  Heavy Rain / Storm" },
  { value: "foggy", label: "🌫️  Foggy" },
  { value: "windy", label: "💨  Windy" },
];

const STATUS_LABELS = {
  active: "Active",
  pending: "Pending",
  completed: "Completed",
  onhold: "On Hold",
};

const STATUS_COLORS = {
  active: { bg: "rgba(34,197,94,0.12)", border: "rgba(34,197,94,0.35)", text: "#86efac", dot: "#22c55e" },
  pending: { bg: "rgba(245,158,11,0.12)", border: "rgba(245,158,11,0.35)", text: "#fbbf24", dot: "#f59e0b" },
  completed: { bg: "rgba(59,130,246,0.12)", border: "rgba(59,130,246,0.35)", text: "#93c5fd", dot: "#3b82f6" },
  onhold: { bg: "rgba(239,68,68,0.12)", border: "rgba(239,68,68,0.35)", text: "#fca5a5", dot: "#ef4444" },
};

const PROGRESS_COLORS = {
  active: "#22c55e",
  pending: "#f59e0b",
  completed: "#3b82f6",
  onhold: "#ef4444",
};

// ============================================================
// HELPERS
// ============================================================
const fmtDate = (d) => d ? new Date(d).toLocaleDateString("en-IN", { day: "2-digit", month: "short", year: "numeric" }) : "—";
const todayISO = () => new Date().toISOString().split("T")[0];

const validateEmail = (e) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(e);

const validateDPR = (d) => {
  const err = {};
  if (!d.projectId) err.projectId = "Please select a project to continue";
  if (!d.date) err.date = "Date is required";
  else if (d.date > todayISO()) err.date = "Date cannot be in the future";
  if (!d.weather) err.weather = "Please select weather conditions";
  if (!d.workDescription || d.workDescription.trim().length < 20)
    err.workDescription = "Work description must be at least 20 characters";
  if (!d.workerCount || isNaN(d.workerCount)) err.workerCount = "Worker count is required";
  else if (Number(d.workerCount) < 1) err.workerCount = "Must be at least 1 worker";
  else if (Number(d.workerCount) > 5000) err.workerCount = "Exceeds maximum (5000)";
  return err;
};

// ============================================================
// THEME
// ============================================================
const darkTheme = {
  bg: "#0d0f12",
  surface: "#13161c",
  card: "#191d26",
  border: "#252b38",
  borderHover: "#3a4257",
  text: "#e2e8f2",
  textSub: "#8694aa",
  textMuted: "#4e5e75",
  amber: "#f59e0b",
  amberLight: "#fbbf24",
  amberGlow: "rgba(245,158,11,0.12)",
  amberBorder: "rgba(245,158,11,0.3)",
  red: "#ef4444",
  redBg: "rgba(239,68,68,0.1)",
  redBorder: "rgba(239,68,68,0.3)",
  redText: "#fca5a5",
  green: "#22c55e",
  input: "#0d0f12",
};

const lightTheme = {
  bg: "#f0f2f6",
  surface: "#ffffff",
  card: "#f8f9fc",
  border: "#dde2ec",
  borderHover: "#b8c1d4",
  text: "#0f1523",
  textSub: "#4a5568",
  textMuted: "#8a96aa",
  amber: "#d97706",
  amberLight: "#f59e0b",
  amberGlow: "rgba(217,119,6,0.08)",
  amberBorder: "rgba(217,119,6,0.25)",
  red: "#dc2626",
  redBg: "rgba(220,38,38,0.08)",
  redBorder: "rgba(220,38,38,0.25)",
  redText: "#dc2626",
  green: "#16a34a",
  input: "#ffffff",
};

// ============================================================
// ICON COMPONENTS
// ============================================================
const Icon = ({ d, size = 16, stroke = "currentColor", sw = 2, fill = "none", viewBox = "0 0 24 24", style }) => (
  <svg width={size} height={size} viewBox={viewBox} fill={fill} stroke={stroke} strokeWidth={sw} strokeLinecap="round" strokeLinejoin="round" style={style}>
    {Array.isArray(d) ? d.map((path, i) => <path key={i} d={path} />) : <path d={d} />}
  </svg>
);

const Icons = {
  layers: ["M12 2 2 7l10 5 10-5-10-5z", "M2 17l10 5 10-5", "M2 12l10 5 10-5"],
  grid: null,
  file: ["M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z", "M14 2v6h6", "M16 13H8", "M16 17H8", "M10 9H8"],
  logout: ["M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4", "M16 17l5-5-5-5", "M21 12H9"],
  sun: ["M12 1v2", "M12 21v2", "M4.22 4.22l1.42 1.42", "M18.36 18.36l1.42 1.42", "M1 12h2", "M21 12h2", "M4.22 19.78l1.42-1.42", "M18.36 5.64l1.42-1.42"],
  moon: "M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z",
  home: ["M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z", "M9 22V12h6v10"],
  mail: ["M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z", "M22 6l-10 7L2 6"],
  lock: ["M19 11H5a2 2 0 0 0-2 2v7a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7a2 2 0 0 0-2-2z", "M7 11V7a5 5 0 0 1 10 0v4"],
  eye: ["M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z", "M12 9a3 3 0 1 0 0 6 3 3 0 0 0 0-6z"],
  eyeOff: ["M17.94 17.94A10.07 10.07 0 0 1 12 20c-7 0-11-8-11-8a18.45 18.45 0 0 1 5.06-5.94M9.9 4.24A9.12 9.12 0 0 1 12 4c7 0 11 8 11 8a18.5 18.5 0 0 1-2.16 3.19m-6.72-1.07a3 3 0 1 1-4.24-4.24", "M1 1l22 22"],
  arrowRight: ["M5 12h14", "M12 5l7 7-7 7"],
  arrowLeft: ["M19 12H5", "M12 19l-7-7 7-7"],
  pin: ["M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0 1 18 0z", "M12 10a2 2 0 1 1 0-4 2 2 0 0 1 0 4"],
  calendar: ["M3 4h18a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2H3a2 2 0 0 1-2-2V6a2 2 0 0 1 2-2z", "M8 2v4", "M16 2v4", "M1 10h22"],
  dollar: ["M12 1v22", "M17 5H9.5a3.5 3.5 0 0 0 0 7h5a3.5 3.5 0 0 1 0 7H6"],
  user: ["M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2", "M12 3a4 4 0 1 0 0 8 4 4 0 0 0 0-8z"],
  search: ["M11 17.25a6.25 6.25 0 1 1 0-12.5 6.25 6.25 0 0 1 0 12.5z", "M16 16l4.5 4.5"],
  x: ["M18 6L6 18", "M6 6l12 12"],
  check: "M20 6L9 17l-5-5",
  upload: ["M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4", "M17 8l-5-5-5 5", "M12 3v12"],
  img: ["M3 3h18a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2H3a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2z", "M8.5 8.5a2 2 0 1 1 0-4 2 2 0 0 1 0 4z", "M21 15l-5-5L5 21"],
  info: ["M12 22a10 10 0 1 1 0-20 10 10 0 0 1 0 20z", "M12 8v4", "M12 16h.01"],
  checkCircle: ["M22 11.08V12a10 10 0 1 1-5.93-9.14", "M22 4L12 14.01l-3-3"],
  warning: ["M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z", "M12 9v4", "M12 17h.01"],
  hardHat: "M2 20h20M6 20v-8a6 6 0 0 1 12 0v8",
  plus: ["M12 5v14", "M5 12h14"],
  filter: ["M22 3H2l8 9.46V19l4 2v-8.54L22 3z"],
};

// ============================================================
// REUSABLE COMPONENTS
// ============================================================

function Toast({ toasts, removeToast }) {
  return (
    <div style={{ position: "fixed", top: 76, right: 16, zIndex: 9999, display: "flex", flexDirection: "column", gap: 8, pointerEvents: "none" }}>
      {toasts.map((t) => (
        <div
          key={t.id}
          onClick={() => removeToast(t.id)}
          style={{
            display: "flex", alignItems: "flex-start", gap: 10,
            padding: "12px 16px", borderRadius: 6, minWidth: 280, maxWidth: 360,
            background: t.type === "success" ? "rgba(34,197,94,0.12)" : t.type === "error" ? "rgba(239,68,68,0.12)" : "rgba(59,130,246,0.12)",
            border: `1px solid ${t.type === "success" ? "rgba(34,197,94,0.4)" : t.type === "error" ? "rgba(239,68,68,0.4)" : "rgba(59,130,246,0.4)"}`,
            color: t.type === "success" ? "#86efac" : t.type === "error" ? "#fca5a5" : "#93c5fd",
            boxShadow: "0 8px 32px rgba(0,0,0,0.5)",
            pointerEvents: "all", cursor: "pointer",
            animation: "toastSlide 0.35s cubic-bezier(0.34,1.56,0.64,1) both",
          }}
        >
          <svg width={18} height={18} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2.5} strokeLinecap="round" strokeLinejoin="round" style={{ flexShrink: 0, marginTop: 1 }}>
            {t.type === "success" ? (
              <><path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"/><path d="M22 4L12 14.01l-3-3"/></>
            ) : t.type === "error" ? (
              <><circle cx="12" cy="12" r="10"/><path d="M12 8v4"/><path d="M12 16h.01"/></>
            ) : (
              <><circle cx="12" cy="12" r="10"/><path d="M12 16v-4"/><path d="M12 8h.01"/></>
            )}
          </svg>
          <p style={{ fontSize: 13.5, fontWeight: 500, lineHeight: 1.4, flex: 1 }}>{t.message}</p>
        </div>
      ))}
    </div>
  );
}

function Badge({ status }) {
  const c = STATUS_COLORS[status] || STATUS_COLORS.pending;
  return (
    <span style={{
      display: "inline-flex", alignItems: "center", gap: 5,
      padding: "3px 8px", borderRadius: 4,
      background: c.bg, border: `1px solid ${c.border}`, color: c.text,
      fontFamily: "'Barlow Condensed', sans-serif", fontSize: 11, fontWeight: 700,
      letterSpacing: "0.1em", textTransform: "uppercase",
    }}>
      <span style={{ width: 5, height: 5, borderRadius: "50%", background: c.dot, display: "inline-block" }} />
      {STATUS_LABELS[status]}
    </span>
  );
}

function FormField({ label, error, children, required }) {
  return (
    <div style={{ display: "flex", flexDirection: "column", gap: 5, marginBottom: 18 }}>
      <label style={{
        fontFamily: "'Barlow Condensed', sans-serif", fontSize: 11, fontWeight: 700,
        letterSpacing: "0.12em", textTransform: "uppercase", color: "var(--text-sub)",
      }}>
        {label}{required && <span style={{ color: "var(--amber)", marginLeft: 2 }}>*</span>}
      </label>
      {children}
      {error && (
        <span style={{ fontSize: 12, color: "var(--red-text)", display: "flex", alignItems: "center", gap: 4 }}>
          <svg width={12} height={12} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2.5} strokeLinecap="round" strokeLinejoin="round">
            <path d="M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"/><path d="M12 9v4"/><path d="M12 17h.01"/>
          </svg>
          {error}
        </span>
      )}
    </div>
  );
}

// ============================================================
// MAIN APP
// ============================================================
export default function App() {
  const [page, setPage] = useState("login"); // login | projects | dpr
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState("dark");
  const [toasts, setToasts] = useState([]);
  const [selectedProjectId, setSelectedProjectId] = useState(null);

  const T = theme === "dark" ? darkTheme : lightTheme;

  const addToast = useCallback((message, type = "success") => {
    const id = Date.now();
    setToasts((p) => [...p, { id, message, type }]);
    setTimeout(() => setToasts((p) => p.filter((t) => t.id !== id)), 4500);
  }, []);

  const removeToast = useCallback((id) => setToasts((p) => p.filter((t) => t.id !== id)), []);

  const navigate = (pg, projectId = null) => {
    setSelectedProjectId(projectId);
    setPage(pg);
  };

  // Inject fonts + keyframes
  useEffect(() => {
    const link = document.createElement("link");
    link.rel = "stylesheet";
    link.href = "https://fonts.googleapis.com/css2?family=Barlow+Condensed:wght@400;500;600;700;800;900&family=Barlow:wght@300;400;500;600&display=swap";
    document.head.appendChild(link);

    const style = document.createElement("style");
    style.textContent = `
      *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
      body { font-family: 'Barlow', sans-serif; -webkit-font-smoothing: antialiased; }
      ::-webkit-scrollbar { width: 4px; }
      ::-webkit-scrollbar-track { background: transparent; }
      ::-webkit-scrollbar-thumb { background: #3a4257; border-radius: 2px; }
      @keyframes fadeUp { from { opacity:0; transform:translateY(14px); } to { opacity:1; transform:translateY(0); } }
      @keyframes scaleIn { from { opacity:0; transform:scale(0.94); } to { opacity:1; transform:scale(1); } }
      @keyframes toastSlide { from { opacity:0; transform:translateX(60px); } to { opacity:1; transform:translateX(0); } }
      @keyframes spin { to { transform: rotate(360deg); } }
      @keyframes shimmer { from{background-position:-200% 0} to{background-position:200% 0} }
      input[type="date"]::-webkit-calendar-picker-indicator { filter: invert(0.6); cursor: pointer; }
      input::placeholder, textarea::placeholder { color: var(--text-muted-ph) !important; }
      select option { background: #13161c; color: #e2e8f2; }
    `;
    document.head.appendChild(style);
    return () => { document.head.removeChild(link); document.head.removeChild(style); };
  }, []);

  const cssVars = {
    "--amber": T.amber,
    "--text-sub": T.textSub,
    "--red-text": T.redText,
    "--text-muted-ph": T.textMuted,
  };

  return (
    <div style={{
      minHeight: "100vh", background: T.bg, color: T.text,
      fontFamily: "'Barlow', sans-serif", ...cssVars,
      position: "relative", overflow: "hidden",
    }}>
      {/* Top accent bar */}
      <div style={{ position: "fixed", top: 0, left: 0, right: 0, height: 3, background: `linear-gradient(90deg, ${T.amber}, ${T.amberLight}, transparent)`, zIndex: 200 }} />

      <Toast toasts={toasts} removeToast={removeToast} />

      {user && page !== "login" && (
        <Navbar user={user} page={page} theme={theme} T={T} setTheme={setTheme}
          navigate={navigate} onLogout={() => { setUser(null); setPage("login"); }} />
      )}

      <div style={{ paddingTop: user && page !== "login" ? 64 : 0 }}>
        {page === "login" && (
          <LoginPage T={T} theme={theme} setTheme={setTheme}
            onLogin={(u) => { setUser(u); setPage("projects"); addToast(`Welcome back, ${u.name}!`); }} />
        )}
        {page === "projects" && (
          <ProjectListPage T={T} navigate={navigate} />
        )}
        {page === "dpr" && (
          <DPRPage T={T} navigate={navigate} selectedProjectId={selectedProjectId}
            addToast={addToast} />
        )}
      </div>
    </div>
  );
}

// ============================================================
// NAVBAR
// ============================================================
function Navbar({ user, page, theme, T, setTheme, navigate, onLogout }) {
  const [menuOpen, setMenuOpen] = useState(false);

  return (
    <nav style={{
      position: "fixed", top: 3, left: 0, right: 0, height: 61, zIndex: 100,
      background: T.surface, borderBottom: `1px solid ${T.border}`,
      display: "flex", alignItems: "center",
    }}>
      <div style={{
        width: "100%", maxWidth: 1280, margin: "0 auto",
        padding: "0 16px", display: "flex", alignItems: "center", gap: 8,
      }}>
        {/* Brand */}
        <button onClick={() => navigate("projects")} style={{
          display: "flex", alignItems: "center", gap: 8, background: "none",
          border: "none", cursor: "pointer", flexShrink: 0, padding: "4px 8px 4px 0",
        }}>
          <div style={{
            width: 32, height: 32, background: T.amber, borderRadius: 4,
            display: "flex", alignItems: "center", justifyContent: "center", color: "#000",
          }}>
            <svg width={18} height={18} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2.5} strokeLinecap="round" strokeLinejoin="round">
              <polygon points="12 2 2 7 12 12 22 7 12 2"/><polyline points="2 17 12 22 22 17"/><polyline points="2 12 12 17 22 12"/>
            </svg>
          </div>
          <span style={{ fontFamily: "'Barlow Condensed', sans-serif", fontSize: 18, fontWeight: 800, letterSpacing: "0.04em", textTransform: "uppercase", color: T.text }}>
            Construct<span style={{ color: T.amber }}>IQ</span>
          </span>
        </button>

        {/* Desktop nav links */}
        <div style={{ display: "flex", gap: 2, flex: 1, "@media(max-width:640px)": { display: "none" } }}>
          {[{ id: "projects", label: "Projects", icon: "grid" }, { id: "dpr", label: "New DPR", icon: "file" }].map((item) => (
            <button key={item.id} onClick={() => navigate(item.id)} style={{
              display: "flex", alignItems: "center", gap: 6,
              padding: "6px 12px", borderRadius: 4, border: "none",
              background: page === item.id ? T.amberGlow : "none",
              color: page === item.id ? T.amber : T.textSub,
              fontFamily: "'Barlow Condensed', sans-serif", fontSize: 12, fontWeight: 700,
              letterSpacing: "0.08em", textTransform: "uppercase", cursor: "pointer",
              transition: "all 0.2s",
              borderBottom: page === item.id ? `2px solid ${T.amber}` : "2px solid transparent",
            }}>
              <svg width={14} height={14} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2} strokeLinecap="round" strokeLinejoin="round">
                {item.id === "projects" ? (
                  <><rect x="3" y="3" width="7" height="7"/><rect x="14" y="3" width="7" height="7"/><rect x="14" y="14" width="7" height="7"/><rect x="3" y="14" width="7" height="7"/></>
                ) : (
                  <><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="16" y1="13" x2="8" y2="13"/><line x1="16" y1="17" x2="8" y2="17"/></>
                )}
              </svg>
              {item.label}
            </button>
          ))}
        </div>

        <div style={{ marginLeft: "auto", display: "flex", alignItems: "center", gap: 8 }}>
          {/* Theme toggle */}
          <button onClick={() => setTheme(t => t === "dark" ? "light" : "dark")} style={{
            width: 34, height: 34, display: "flex", alignItems: "center", justifyContent: "center",
            background: T.card, border: `1px solid ${T.border}`, borderRadius: 4,
            color: T.textSub, cursor: "pointer", transition: "all 0.2s", flexShrink: 0,
          }}>
            {theme === "dark" ? (
              <svg width={15} height={15} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2} strokeLinecap="round" strokeLinejoin="round">
                <circle cx="12" cy="12" r="5"/><line x1="12" y1="1" x2="12" y2="3"/><line x1="12" y1="21" x2="12" y2="23"/><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"/><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"/><line x1="1" y1="12" x2="3" y2="12"/><line x1="21" y1="12" x2="23" y2="12"/><line x1="4.22" y1="19.78" x2="5.64" y2="18.36"/><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"/>
              </svg>
            ) : (
              <svg width={15} height={15} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2} strokeLinecap="round" strokeLinejoin="round">
                <path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"/>
              </svg>
            )}
          </button>

          {/* User pill */}
          <div style={{ display: "flex", alignItems: "center", gap: 8 }}>
            <div style={{
              width: 32, height: 32, background: T.amber, color: "#000",
              borderRadius: 4, display: "flex", alignItems: "center", justifyContent: "center",
              fontFamily: "'Barlow Condensed', sans-serif", fontSize: 12, fontWeight: 900,
              letterSpacing: "0.05em", flexShrink: 0,
            }}>{user.avatar}</div>
            <div style={{ display: "none", flexDirection: "column", lineHeight: 1.2 }} className="user-info-desktop">
              <span style={{ fontFamily: "'Barlow Condensed', sans-serif", fontSize: 13, fontWeight: 700, color: T.text }}>{user.name}</span>
              <span style={{ fontSize: 11, color: T.textMuted }}>{user.role}</span>
            </div>
          </div>

          {/* Logout */}
          <button onClick={onLogout} style={{
            display: "flex", alignItems: "center", gap: 6,
            padding: "6px 12px", background: "none",
            border: `1px solid ${T.border}`, borderRadius: 4,
            color: T.textSub, fontFamily: "'Barlow Condensed', sans-serif",
            fontSize: 11, fontWeight: 700, letterSpacing: "0.08em", textTransform: "uppercase",
            cursor: "pointer", transition: "all 0.2s",
          }}>
            <svg width={13} height={13} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2} strokeLinecap="round" strokeLinejoin="round">
              <path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"/><polyline points="16 17 21 12 16 7"/><line x1="21" y1="12" x2="9" y2="12"/>
            </svg>
            <span style={{ display: "none" }}>Logout</span>
          </button>
        </div>
      </div>
    </nav>
  );
}

// ============================================================
// LOGIN PAGE
// ============================================================
function LoginPage({ T, theme, setTheme, onLogin }) {
  const [form, setForm] = useState({ email: "", password: "" });
  const [errors, setErrors] = useState({});
  const [loading, setLoading] = useState(false);
  const [authErr, setAuthErr] = useState("");
  const [showPw, setShowPw] = useState(false);
  const [mounted, setMounted] = useState(false);

  useEffect(() => { setTimeout(() => setMounted(true), 50); }, []);

  const validate = () => {
    const e = {};
    if (!form.email) e.email = "Email address is required";
    else if (!validateEmail(form.email)) e.email = "Please enter a valid email address";
    if (!form.password) e.password = "Password is required";
    else if (form.password.length < 4) e.password = "Password must be at least 4 characters";
    return e;
  };

  const handleSubmit = async (ev) => {
    ev.preventDefault();
    setAuthErr("");
    const errs = validate();
    if (Object.keys(errs).length) { setErrors(errs); return; }
    setErrors({});
    setLoading(true);
    await new Promise(r => setTimeout(r, 900));
    setLoading(false);
    if (form.email === "test@test.com" && form.password === "123456") {
      onLogin({ email: form.email, name: "Alex Morgan", role: "Site Manager", avatar: "AM" });
    } else {
      setAuthErr("Invalid credentials. Please use test@test.com / 123456");
    }
  };

  const fillDemo = () => {
    setForm({ email: "test@test.com", password: "123456" });
    setErrors({}); setAuthErr("");
  };

  const inputStyle = (hasErr) => ({
    width: "100%", padding: "11px 14px", paddingLeft: 40,
    background: T.input, border: `1.5px solid ${hasErr ? T.red : T.border}`,
    borderRadius: 5, color: T.text, fontFamily: "'Barlow', sans-serif",
    fontSize: 14.5, outline: "none", transition: "border-color 0.2s, box-shadow 0.2s",
    boxShadow: hasErr ? `0 0 0 3px ${T.redBg}` : "none",
  });

  return (
    <div style={{
      minHeight: "100vh", display: "flex", alignItems: "center", justifyContent: "center",
      padding: "24px 16px", position: "relative", overflow: "hidden", background: T.bg,
    }}>
      {/* Background grid */}
      <div style={{
        position: "absolute", inset: 0, opacity: theme === "dark" ? 0.25 : 0.12,
        backgroundImage: `linear-gradient(${T.border} 1px, transparent 1px), linear-gradient(90deg, ${T.border} 1px, transparent 1px)`,
        backgroundSize: "44px 44px", pointerEvents: "none",
      }} />
      {/* Glow */}
      <div style={{
        position: "absolute", top: "-15%", left: "-5%", width: "50%", height: "50%",
        background: `radial-gradient(ellipse, ${T.amberGlow} 0%, transparent 70%)`,
        pointerEvents: "none",
      }} />

      {/* Theme toggle */}
      <button onClick={() => setTheme(t => t === "dark" ? "light" : "dark")} style={{
        position: "fixed", top: 16, right: 16, zIndex: 10,
        width: 36, height: 36, display: "flex", alignItems: "center", justifyContent: "center",
        background: T.surface, border: `1px solid ${T.border}`, borderRadius: 4,
        color: T.textSub, cursor: "pointer",
      }}>
        {theme === "dark" ? (
          <svg width={15} height={15} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2} strokeLinecap="round" strokeLinejoin="round">
            <circle cx="12" cy="12" r="5"/><line x1="12" y1="1" x2="12" y2="3"/><line x1="12" y1="21" x2="12" y2="23"/><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"/><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"/><line x1="1" y1="12" x2="3" y2="12"/><line x1="21" y1="12" x2="23" y2="12"/>
          </svg>
        ) : (
          <svg width={15} height={15} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2} strokeLinecap="round" strokeLinejoin="round">
            <path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"/>
          </svg>
        )}
      </button>

      <div style={{
        width: "100%", maxWidth: 420, position: "relative", zIndex: 1,
        opacity: mounted ? 1 : 0, transform: mounted ? "scale(1)" : "scale(0.95)",
        transition: "opacity 0.4s ease, transform 0.4s ease",
      }}>
        {/* Card */}
        <div style={{
          background: T.surface, border: `1px solid ${T.border}`,
          borderRadius: 8, overflow: "hidden", boxShadow: "0 24px 64px rgba(0,0,0,0.4)",
        }}>
          {/* Header */}
          <div style={{
            padding: "24px 28px 20px", borderBottom: `1px solid ${T.border}`,
            background: T.card, display: "flex", alignItems: "center", gap: 12,
          }}>
            <div style={{
              width: 40, height: 40, background: T.amber, borderRadius: 6,
              display: "flex", alignItems: "center", justifyContent: "center", color: "#000", flexShrink: 0,
            }}>
              <svg width={22} height={22} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2.5} strokeLinecap="round" strokeLinejoin="round">
                <polygon points="12 2 2 7 12 12 22 7 12 2"/><polyline points="2 17 12 22 22 17"/><polyline points="2 12 12 17 22 12"/>
              </svg>
            </div>
            <div>
              <div style={{ fontFamily: "'Barlow Condensed', sans-serif", fontSize: 22, fontWeight: 900, letterSpacing: "0.04em", textTransform: "uppercase", lineHeight: 1 }}>
                Construct<span style={{ color: T.amber }}>IQ</span>
              </div>
              <div style={{ fontSize: 11, color: T.textMuted, letterSpacing: "0.08em", textTransform: "uppercase", fontFamily: "'Barlow Condensed', sans-serif", marginTop: 2 }}>
                Field Management Platform
              </div>
            </div>
          </div>

          {/* Body */}
          <div style={{ padding: "24px 28px 28px" }}>
            <div style={{ display: "flex", alignItems: "center", justifyContent: "space-between", marginBottom: 20 }}>
              <h2 style={{ fontFamily: "'Barlow Condensed', sans-serif", fontSize: 20, fontWeight: 800, letterSpacing: "0.03em", textTransform: "uppercase" }}>
                Sign In
              </h2>
              <button onClick={fillDemo} style={{
                background: "none", border: "none", color: T.amber, fontSize: 12,
                cursor: "pointer", textDecoration: "underline", textUnderlineOffset: 3, fontFamily: "'Barlow', sans-serif",
              }}>
                Use demo credentials
              </button>
            </div>

            {authErr && (
              <div style={{
                display: "flex", alignItems: "center", gap: 8,
                padding: "10px 14px", marginBottom: 18,
                background: T.redBg, border: `1px solid ${T.redBorder}`,
                borderRadius: 5, color: T.redText, fontSize: 13.5,
                animation: "fadeUp 0.3s ease both",
              }}>
                <svg width={15} height={15} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2} strokeLinecap="round" strokeLinejoin="round" style={{ flexShrink: 0 }}>
                  <circle cx="12" cy="12" r="10"/><path d="M12 8v4"/><path d="M12 16h.01"/>
                </svg>
                {authErr}
              </div>
            )}

            <form onSubmit={handleSubmit} noValidate>
              {/* Email */}
              <FormField label="Email Address" error={errors.email} required>
                <div style={{ position: "relative" }}>
                  <svg width={15} height={15} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2} strokeLinecap="round" strokeLinejoin="round"
                    style={{ position: "absolute", left: 12, top: "50%", transform: "translateY(-50%)", color: T.textMuted, pointerEvents: "none" }}>
                    <path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"/><polyline points="22,6 12,13 2,6"/>
                  </svg>
                  <input type="email" value={form.email} placeholder="you@company.com"
                    onChange={e => { setForm(f => ({ ...f, email: e.target.value })); setErrors(er => ({ ...er, email: "" })); setAuthErr(""); }}
                    style={inputStyle(!!errors.email)}
                    onFocus={e => { if (!errors.email) e.target.style.borderColor = T.amber; e.target.style.boxShadow = `0 0 0 3px ${T.amberGlow}`; }}
                    onBlur={e => { e.target.style.borderColor = errors.email ? T.red : T.border; e.target.style.boxShadow = errors.email ? `0 0 0 3px ${T.redBg}` : "none"; }}
                  />
                </div>
              </FormField>

              {/* Password */}
              <FormField label="Password" error={errors.password} required>
                <div style={{ position: "relative" }}>
                  <svg width={15} height={15} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2} strokeLinecap="round" strokeLinejoin="round"
                    style={{ position: "absolute", left: 12, top: "50%", transform: "translateY(-50%)", color: T.textMuted, pointerEvents: "none" }}>
                    <rect x="3" y="11" width="18" height="11" rx="2" ry="2"/><path d="M7 11V7a5 5 0 0 1 10 0v4"/>
                  </svg>
                  <input type={showPw ? "text" : "password"} value={form.password} placeholder="Enter your password"
                    onChange={e => { setForm(f => ({ ...f, password: e.target.value })); setErrors(er => ({ ...er, password: "" })); setAuthErr(""); }}
                    style={{ ...inputStyle(!!errors.password), paddingRight: 42 }}
                    onFocus={e => { if (!errors.password) e.target.style.borderColor = T.amber; e.target.style.boxShadow = `0 0 0 3px ${T.amberGlow}`; }}
                    onBlur={e => { e.target.style.borderColor = errors.password ? T.red : T.border; e.target.style.boxShadow = errors.password ? `0 0 0 3px ${T.redBg}` : "none"; }}
                  />
                  <button type="button" onClick={() => setShowPw(v => !v)} style={{
                    position: "absolute", right: 12, top: "50%", transform: "translateY(-50%)",
                    background: "none", border: "none", color: T.textMuted, cursor: "pointer", display: "flex",
                  }}>
                    <svg width={15} height={15} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2} strokeLinecap="round" strokeLinejoin="round">
                      {showPw ? (
                        <><path d="M17.94 17.94A10.07 10.07 0 0 1 12 20c-7 0-11-8-11-8a18.45 18.45 0 0 1 5.06-5.94M9.9 4.24A9.12 9.12 0 0 1 12 4c7 0 11 8 11 8a18.5 18.5 0 0 1-2.16 3.19m-6.72-1.07a3 3 0 1 1-4.24-4.24"/><line x1="1" y1="1" x2="23" y2="23"/></>
                      ) : (
                        <><path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"/><circle cx="12" cy="12" r="3"/></>
                      )}
                    </svg>
                  </button>
                </div>
              </FormField>

              <button type="submit" disabled={loading} style={{
                width: "100%", padding: "12px 20px", marginTop: 4,
                background: loading ? T.amberGlow : T.amber,
                border: loading ? `1px solid ${T.amberBorder}` : "none",
                borderRadius: 5, color: loading ? T.amber : "#000",
                fontFamily: "'Barlow Condensed', sans-serif", fontSize: 14, fontWeight: 800,
                letterSpacing: "0.1em", textTransform: "uppercase", cursor: loading ? "not-allowed" : "pointer",
                display: "flex", alignItems: "center", justifyContent: "center", gap: 8,
                transition: "all 0.2s", opacity: loading ? 0.8 : 1,
              }}>
                {loading ? (
                  <><div style={{ width: 16, height: 16, border: `2px solid ${T.amberBorder}`, borderTopColor: T.amber, borderRadius: "50%", animation: "spin 0.6s linear infinite" }} />Authenticating...</>
                ) : (
                  <>Sign In to Dashboard <svg width={16} height={16} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2.5} strokeLinecap="round" strokeLinejoin="round"><line x1="5" y1="12" x2="19" y2="12"/><polyline points="12 5 19 12 12 19"/></svg></>
                )}
              </button>
            </form>
          </div>

          {/* Footer */}
          <div style={{
            padding: "12px 28px", borderTop: `1px solid ${T.border}`,
            background: T.card, textAlign: "center",
          }}>
            <p style={{ fontSize: 12, color: T.textMuted }}>
              Demo credentials: <code style={{ color: T.amber, background: T.amberGlow, padding: "1px 6px", borderRadius: 3, fontFamily: "monospace" }}>test@test.com</code> / <code style={{ color: T.amber, background: T.amberGlow, padding: "1px 6px", borderRadius: 3, fontFamily: "monospace" }}>123456</code>
            </p>
          </div>
        </div>

        {/* Stats row below card */}
        <div style={{ display: "flex", gap: 10, marginTop: 12 }}>
          {[{ n: "5", l: "Projects" }, { n: "1,240", l: "Workers" }, { n: "98.2%", l: "Safety" }].map((s, i) => (
            <div key={i} style={{
              flex: 1, background: T.surface, border: `1px solid ${T.border}`,
              borderRadius: 6, padding: "10px 12px", display: "flex", flexDirection: "column", gap: 2,
              opacity: mounted ? 1 : 0, transform: mounted ? "translateY(0)" : "translateY(10px)",
              transition: `opacity 0.4s ease ${0.15 + i * 0.08}s, transform 0.4s ease ${0.15 + i * 0.08}s`,
            }}>
              <span style={{ fontFamily: "'Barlow Condensed', sans-serif", fontSize: 22, fontWeight: 900, color: T.amber, lineHeight: 1 }}>{s.n}</span>
              <span style={{ fontFamily: "'Barlow Condensed', sans-serif", fontSize: 10, color: T.textMuted, textTransform: "uppercase", letterSpacing: "0.08em" }}>{s.l}</span>
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}

// ============================================================
// PROJECT LIST PAGE
// ============================================================
function ProjectListPage({ T, navigate }) {
  const [search, setSearch] = useState("");
  const [statusFilter, setStatusFilter] = useState("all");
  const [mounted, setMounted] = useState(false);

  useEffect(() => { setTimeout(() => setMounted(true), 50); }, []);

  const filtered = PROJECTS.filter(p => {
    const ms = !search || [p.name, p.location, p.client, p.id].some(v => v.toLowerCase().includes(search.toLowerCase()));
    const mf = statusFilter === "all" || p.status === statusFilter;
    return ms && mf;
  });

  const counts = { all: PROJECTS.length, ...Object.fromEntries(["active","pending","completed","onhold"].map(s => [s, PROJECTS.filter(p => p.status === s).length])) };

  const FILTERS = [
    { id: "all", label: "All", count: counts.all },
    { id: "active", label: "Active", count: counts.active },
    { id: "pending", label: "Pending", count: counts.pending },
    { id: "completed", label: "Completed", count: counts.completed },
    { id: "onhold", label: "On Hold", count: counts.onhold },
  ];

  return (
    <div style={{ minHeight: "calc(100vh - 64px)", padding: "24px 0 48px", background: T.bg }}>
      <div style={{ maxWidth: 1280, margin: "0 auto", padding: "0 16px" }}>
        {/* Header */}
        <div style={{
          display: "flex", alignItems: "flex-end", justifyContent: "space-between",
          gap: 12, marginBottom: 24,
          opacity: mounted ? 1 : 0, transform: mounted ? "translateY(0)" : "translateY(12px)",
          transition: "all 0.4s ease",
        }}>
          <div>
            <div style={{ display: "flex", alignItems: "center", gap: 5, fontSize: 11, color: T.textMuted, fontFamily: "'Barlow Condensed', sans-serif", textTransform: "uppercase", letterSpacing: "0.1em", marginBottom: 6 }}>
              <svg width={11} height={11} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2} strokeLinecap="round" strokeLinejoin="round">
                <path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"/><polyline points="9 22 9 12 15 12 15 22"/>
              </svg>
              Dashboard
            </div>
            <h1 style={{
              fontFamily: "'Barlow Condensed', sans-serif", fontSize: "clamp(22px, 4vw, 32px)",
              fontWeight: 900, letterSpacing: "0.02em", textTransform: "uppercase", lineHeight: 1,
              display: "flex", alignItems: "center", gap: 10,
            }}>
              Project Portfolio
              <span style={{
                background: T.amberGlow, border: `1px solid ${T.amberBorder}`, color: T.amber,
                fontFamily: "'Barlow Condensed', sans-serif", fontSize: 14, fontWeight: 700,
                padding: "2px 10px", borderRadius: 4,
              }}>{filtered.length}</span>
            </h1>
          </div>
          <button onClick={() => navigate("dpr")} style={{
            display: "flex", alignItems: "center", gap: 7, padding: "9px 18px",
            background: T.amber, border: "none", borderRadius: 5, color: "#000",
            fontFamily: "'Barlow Condensed', sans-serif", fontSize: 13, fontWeight: 800,
            letterSpacing: "0.1em", textTransform: "uppercase", cursor: "pointer",
            transition: "all 0.2s", flexShrink: 0,
          }}>
            <svg width={15} height={15} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2.5} strokeLinecap="round" strokeLinejoin="round">
              <line x1="12" y1="5" x2="12" y2="19"/><line x1="5" y1="12" x2="19" y2="12"/>
            </svg>
            New DPR
          </button>
        </div>

        {/* Status summary pills */}
        <div style={{
          display: "flex", gap: 8, flexWrap: "wrap", marginBottom: 16,
          opacity: mounted ? 1 : 0, transition: "opacity 0.4s ease 0.08s",
        }}>
          {["active","pending","completed","onhold"].map(s => {
            const c = STATUS_COLORS[s];
            return (
              <button key={s} onClick={() => setStatusFilter(f => f === s ? "all" : s)} style={{
                display: "flex", alignItems: "center", gap: 6, padding: "6px 12px",
                background: statusFilter === s ? c.bg : T.card,
                border: `1px solid ${statusFilter === s ? c.border : T.border}`,
                borderRadius: 4, color: statusFilter === s ? c.text : T.textSub,
                fontFamily: "'Barlow Condensed', sans-serif", fontSize: 11, fontWeight: 700,
                letterSpacing: "0.08em", textTransform: "uppercase", cursor: "pointer", transition: "all 0.2s",
              }}>
                <span style={{ width: 6, height: 6, borderRadius: "50%", background: c.dot }} />
                <span style={{ fontWeight: 900, fontSize: 13 }}>{counts[s]}</span>
                {STATUS_LABELS[s]}
              </button>
            );
          })}
        </div>

        {/* Search + filter bar */}
        <div style={{
          display: "flex", flexWrap: "wrap", gap: 10, marginBottom: 20, alignItems: "center",
          opacity: mounted ? 1 : 0, transition: "opacity 0.4s ease 0.12s",
        }}>
          <div style={{ position: "relative", flex: "1 1 240px", maxWidth: 420 }}>
            <svg width={14} height={14} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2} strokeLinecap="round" strokeLinejoin="round"
              style={{ position: "absolute", left: 11, top: "50%", transform: "translateY(-50%)", color: T.textMuted, pointerEvents: "none" }}>
              <circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/>
            </svg>
            <input type="text" placeholder="Search projects, locations, clients…" value={search} onChange={e => setSearch(e.target.value)} style={{
              width: "100%", padding: "8px 36px 8px 34px",
              background: T.card, border: `1px solid ${T.border}`, borderRadius: 5,
              color: T.text, fontFamily: "'Barlow', sans-serif", fontSize: 13.5, outline: "none",
            }} onFocus={e => e.target.style.borderColor = T.amber} onBlur={e => e.target.style.borderColor = T.border} />
            {search && (
              <button onClick={() => setSearch("")} style={{
                position: "absolute", right: 10, top: "50%", transform: "translateY(-50%)",
                background: "none", border: "none", color: T.textMuted, cursor: "pointer", display: "flex",
              }}>
                <svg width={13} height={13} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2.5} strokeLinecap="round" strokeLinejoin="round">
                  <line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/>
                </svg>
              </button>
            )}
          </div>
          <div style={{ display: "flex", gap: 4, flexWrap: "wrap" }}>
            {FILTERS.map(f => (
              <button key={f.id} onClick={() => setStatusFilter(f.id)} style={{
                padding: "6px 12px", background: statusFilter === f.id ? T.amberGlow : "none",
                border: `1px solid ${statusFilter === f.id ? T.amberBorder : T.border}`,
                borderRadius: 4, color: statusFilter === f.id ? T.amber : T.textSub,
                fontFamily: "'Barlow Condensed', sans-serif", fontSize: 11, fontWeight: 700,
                letterSpacing: "0.08em", textTransform: "uppercase", cursor: "pointer", transition: "all 0.2s",
              }}>
                {f.label}
              </button>
            ))}
          </div>
        </div>

        {/* Grid */}
        {filtered.length === 0 ? (
          <div style={{
            display: "flex", flexDirection: "column", alignItems: "center", gap: 12,
            padding: "60px 20px", color: T.textMuted, textAlign: "center",
          }}>
            <svg width={44} height={44} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={1.5} strokeLinecap="round" strokeLinejoin="round">
              <circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/>
            </svg>
            <h3 style={{ fontFamily: "'Barlow Condensed', sans-serif", fontSize: 18, fontWeight: 700, textTransform: "uppercase", letterSpacing: "0.05em", color: T.textSub }}>No projects found</h3>
            <p style={{ fontSize: 13.5 }}>Try adjusting search or filters</p>
            <button onClick={() => { setSearch(""); setStatusFilter("all"); }} style={{
              padding: "8px 16px", background: "none", border: `1px solid ${T.border}`,
              borderRadius: 4, color: T.textSub, fontFamily: "'Barlow Condensed', sans-serif",
              fontSize: 12, fontWeight: 700, letterSpacing: "0.08em", textTransform: "uppercase", cursor: "pointer",
            }}>Clear Filters</button>
          </div>
        ) : (
          <div style={{
            display: "grid",
            gridTemplateColumns: "repeat(auto-fill, minmax(min(100%, 360px), 1fr))",
            gap: 14,
          }}>
            {filtered.map((p, i) => (
              <ProjectCard key={p.id} project={p} index={i} T={T} mounted={mounted}
                onSelect={() => navigate("dpr", p.id)} />
            ))}
          </div>
        )}
      </div>
    </div>
  );
}

function ProjectCard({ project: p, index, T, mounted, onSelect }) {
  const [hovered, setHovered] = useState(false);
  const pc = STATUS_COLORS[p.status];

  return (
    <div onClick={onSelect} onMouseEnter={() => setHovered(true)} onMouseLeave={() => setHovered(false)}
      tabIndex={0} role="button" onKeyDown={e => e.key === "Enter" && onSelect()}
      style={{
        background: T.card, border: `1px solid ${hovered ? pc.border : T.border}`,
        borderRadius: 7, overflow: "hidden", cursor: "pointer",
        transform: mounted ? (hovered ? "translateY(-3px)" : "translateY(0)") : "translateY(14px)",
        opacity: mounted ? 1 : 0,
        transition: `opacity 0.4s ease ${index * 0.06}s, transform 0.25s ease, border-color 0.2s, box-shadow 0.25s`,
        boxShadow: hovered ? `0 8px 28px rgba(0,0,0,0.35), 0 0 0 1px ${pc.border}` : "none",
        outline: "none",
      }}>
      {/* Progress bar top */}
      <div style={{ height: 3, background: T.border }}>
        <div style={{ height: "100%", width: `${p.progress}%`, background: PROGRESS_COLORS[p.status], transition: "width 0.6s ease", borderRadius: "0 2px 2px 0" }} />
      </div>

      <div style={{ padding: "16px 18px" }}>
        {/* Header row */}
        <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 10 }}>
          <span style={{ fontFamily: "'Barlow Condensed', sans-serif", fontSize: 10, color: T.textMuted, letterSpacing: "0.1em", textTransform: "uppercase" }}>{p.id}</span>
          <Badge status={p.status} />
        </div>

        {/* Name + desc */}
        <h3 style={{ fontFamily: "'Barlow Condensed', sans-serif", fontSize: 17, fontWeight: 800, letterSpacing: "0.02em", marginBottom: 5, lineHeight: 1.2 }}>
          {p.name}
        </h3>
        <p style={{ fontSize: 12.5, color: T.textMuted, lineHeight: 1.5, marginBottom: 14,
          display: "-webkit-box", WebkitLineClamp: 2, WebkitBoxOrient: "vertical", overflow: "hidden" }}>
          {p.description}
        </p>

        {/* Progress row */}
        <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 5, alignItems: "center" }}>
          <span style={{ fontFamily: "'Barlow Condensed', sans-serif", fontSize: 10, color: T.textMuted, textTransform: "uppercase", letterSpacing: "0.1em" }}>Completion</span>
          <span style={{ fontFamily: "'Barlow Condensed', sans-serif", fontSize: 13, fontWeight: 800, color: pc.text }}>{p.progress}%</span>
        </div>
        <div style={{ height: 4, background: T.border, borderRadius: 2, marginBottom: 14 }}>
          <div style={{ height: "100%", width: `${p.progress}%`, background: PROGRESS_COLORS[p.status], borderRadius: 2, transition: "width 0.5s ease" }} />
        </div>

        {/* Meta grid */}
        <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: "6px 8px", padding: "10px 0", borderTop: `1px solid ${T.border}`, borderBottom: `1px solid ${T.border}`, marginBottom: 12 }}>
          {[
            { icon: "pin", val: p.location },
            { icon: "calendar", val: fmtDate(p.startDate) },
            { icon: "dollar", val: p.budget },
            { icon: "user", val: p.manager },
          ].map((m, i) => (
            <div key={i} style={{ display: "flex", alignItems: "flex-start", gap: 5, fontSize: 11.5, color: T.textSub }}>
              <svg width={12} height={12} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2} strokeLinecap="round" strokeLinejoin="round" style={{ flexShrink: 0, marginTop: 1.5, color: T.textMuted }}>
                {m.icon === "pin" && <><path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0 1 18 0z"/><circle cx="12" cy="10" r="3"/></>}
                {m.icon === "calendar" && <><rect x="3" y="4" width="18" height="18" rx="2" ry="2"/><line x1="16" y1="2" x2="16" y2="6"/><line x1="8" y1="2" x2="8" y2="6"/><line x1="3" y1="10" x2="21" y2="10"/></>}
                {m.icon === "dollar" && <><line x1="12" y1="1" x2="12" y2="23"/><path d="M17 5H9.5a3.5 3.5 0 0 0 0 7h5a3.5 3.5 0 0 1 0 7H6"/></>}
                {m.icon === "user" && <><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/></>}
              </svg>
              <span style={{ lineHeight: 1.3 }}>{m.val}</span>
            </div>
          ))}
        </div>

        {/* Footer */}
        <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center" }}>
          <span style={{ fontSize: 11.5, color: T.textMuted, display: "flex", alignItems: "center", gap: 4 }}>
            <svg width={11} height={11} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2} strokeLinecap="round" strokeLinejoin="round">
              <path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/>
            </svg>
            {p.client}
          </span>
          <span style={{
            display: "flex", alignItems: "center", gap: 4,
            fontFamily: "'Barlow Condensed', sans-serif", fontSize: 11, fontWeight: 800,
            letterSpacing: "0.1em", textTransform: "uppercase", color: T.amber,
          }}>
            File DPR
            <svg width={12} height={12} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2.5} strokeLinecap="round" strokeLinejoin="round">
              <line x1="5" y1="12" x2="19" y2="12"/><polyline points="12 5 19 12 12 19"/>
            </svg>
          </span>
        </div>
      </div>
    </div>
  );
}

// ============================================================
// DPR FORM PAGE
// ============================================================
function DPRPage({ T, navigate, selectedProjectId, addToast }) {
  const [form, setForm] = useState({
    projectId: selectedProjectId || "",
    date: todayISO(),
    weather: "",
    workDescription: "",
    workerCount: "",
    supervisorName: "",
    remarks: "",
  });
  const [errors, setErrors] = useState({});
  const [photos, setPhotos] = useState([]);
  const [submitting, setSubmitting] = useState(false);
  const [submitted, setSubmitted] = useState(false);
  const [mounted, setMounted] = useState(false);
  const fileRef = useRef();

  useEffect(() => { setTimeout(() => setMounted(true), 50); }, []);

  const set = (k) => (e) => {
    setForm(f => ({ ...f, [k]: e.target.value }));
    if (errors[k]) setErrors(er => ({ ...er, [k]: "" }));
  };

  const handlePhotos = (e) => {
    const files = Array.from(e.target.files || []);
    const remaining = 3 - photos.length;
    const toAdd = files.slice(0, remaining);
    if (toAdd.length < files.length) addToast("Maximum 3 photos allowed", "info");
    toAdd.forEach(file => {
      const reader = new FileReader();
      reader.onload = ev => {
        setPhotos(p => [...p, { id: Date.now() + Math.random(), name: file.name, size: file.size, url: ev.target.result }]);
      };
      reader.readAsDataURL(file);
    });
    e.target.value = "";
  };

  const removePhoto = (id) => setPhotos(p => p.filter(x => x.id !== id));

  const handleSubmit = async (e) => {
    e.preventDefault();
    const errs = validateDPR(form);
    if (Object.keys(errs).length) { setErrors(errs); addToast("Please fix the errors below", "error"); return; }
    setErrors({});
    setSubmitting(true);
    await new Promise(r => setTimeout(r, 1200));
    setSubmitting(false);
    setSubmitted(true);
    addToast("Daily Progress Report submitted successfully! ✓", "success");
  };

  const handleReset = () => {
    setForm({ projectId: "", date: todayISO(), weather: "", workDescription: "", workerCount: "", supervisorName: "", remarks: "" });
    setPhotos([]); setErrors({}); setSubmitted(false);
  };

  const selectedProject = PROJECTS.find(p => p.id === form.projectId);

  const inputStyle = (hasErr) => ({
    width: "100%", padding: "10px 14px",
    background: T.input, border: `1.5px solid ${hasErr ? T.red : T.border}`,
    borderRadius: 5, color: T.text, fontFamily: "'Barlow', sans-serif",
    fontSize: 14, outline: "none", transition: "border-color 0.2s, box-shadow 0.2s",
    boxShadow: hasErr ? `0 0 0 3px ${T.redBg}` : "none",
    appearance: "none", WebkitAppearance: "none",
  });

  const focusHandlers = (field) => ({
    onFocus: (e) => { if (!errors[field]) e.target.style.borderColor = T.amber; e.target.style.boxShadow = `0 0 0 3px ${T.amberGlow}`; },
    onBlur: (e) => { e.target.style.borderColor = errors[field] ? T.red : T.border; e.target.style.boxShadow = errors[field] ? `0 0 0 3px ${T.redBg}` : "none"; },
  });

  if (submitted) {
    return (
      <div style={{ minHeight: "calc(100vh - 64px)", display: "flex", alignItems: "center", justifyContent: "center", padding: 24, background: T.bg }}>
        <div style={{
          background: T.surface, border: `1px solid ${T.border}`, borderRadius: 8,
          padding: "48px 40px", maxWidth: 480, width: "100%", textAlign: "center",
          animation: "scaleIn 0.4s cubic-bezier(0.34,1.56,0.64,1) both",
        }}>
          <div style={{
            width: 72, height: 72, background: "rgba(34,197,94,0.12)", borderRadius: "50%",
            display: "flex", alignItems: "center", justifyContent: "center", margin: "0 auto 20px",
            border: "2px solid rgba(34,197,94,0.3)",
          }}>
            <svg width={36} height={36} viewBox="0 0 24 24" fill="none" stroke="#86efac" strokeWidth={2.5} strokeLinecap="round" strokeLinejoin="round">
              <path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"/><path d="M22 4L12 14.01l-3-3"/>
            </svg>
          </div>
          <h2 style={{ fontFamily: "'Barlow Condensed', sans-serif", fontSize: 26, fontWeight: 900, textTransform: "uppercase", letterSpacing: "0.03em", marginBottom: 8 }}>
            DPR Submitted!
          </h2>
          <p style={{ fontSize: 14, color: T.textSub, marginBottom: 6 }}>
            Daily Progress Report filed successfully for
          </p>
          <p style={{ fontFamily: "'Barlow Condensed', sans-serif", fontSize: 16, fontWeight: 700, color: T.amber, marginBottom: 28 }}>
            {selectedProject?.name || "the selected project"}
          </p>
          <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 8, marginBottom: 28, padding: "16px", background: T.card, borderRadius: 6, border: `1px solid ${T.border}` }}>
            {[
              { l: "Date", v: fmtDate(form.date) },
              { l: "Weather", v: WEATHER_OPTIONS.find(w => w.value === form.weather)?.label || form.weather },
              { l: "Workers", v: form.workerCount },
              { l: "Photos", v: `${photos.length} attached` },
            ].map(item => (
              <div key={item.l} style={{ textAlign: "left" }}>
                <div style={{ fontFamily: "'Barlow Condensed', sans-serif", fontSize: 10, color: T.textMuted, textTransform: "uppercase", letterSpacing: "0.1em", marginBottom: 2 }}>{item.l}</div>
                <div style={{ fontSize: 13.5, fontWeight: 600, color: T.text }}>{item.v}</div>
              </div>
            ))}
          </div>
          <div style={{ display: "flex", gap: 10 }}>
            <button onClick={handleReset} style={{
              flex: 1, padding: "10px", background: "none", border: `1px solid ${T.border}`,
              borderRadius: 5, color: T.textSub, fontFamily: "'Barlow Condensed', sans-serif",
              fontSize: 12, fontWeight: 700, letterSpacing: "0.1em", textTransform: "uppercase", cursor: "pointer",
            }}>New DPR</button>
            <button onClick={() => navigate("projects")} style={{
              flex: 1, padding: "10px", background: T.amber, border: "none",
              borderRadius: 5, color: "#000", fontFamily: "'Barlow Condensed', sans-serif",
              fontSize: 12, fontWeight: 800, letterSpacing: "0.1em", textTransform: "uppercase", cursor: "pointer",
            }}>← Back to Projects</button>
          </div>
        </div>
      </div>
    );
  }

  return (
    <div style={{ minHeight: "calc(100vh - 64px)", padding: "24px 0 48px", background: T.bg }}>
      <div style={{ maxWidth: 860, margin: "0 auto", padding: "0 16px" }}>
        {/* Header */}
        <div style={{
          marginBottom: 24, opacity: mounted ? 1 : 0,
          transform: mounted ? "translateY(0)" : "translateY(12px)",
          transition: "all 0.4s ease",
        }}>
          <button onClick={() => navigate("projects")} style={{
            display: "flex", alignItems: "center", gap: 5, background: "none", border: "none",
            color: T.textMuted, fontSize: 12, fontFamily: "'Barlow Condensed', sans-serif",
            letterSpacing: "0.08em", textTransform: "uppercase", cursor: "pointer", marginBottom: 8, padding: 0,
          }}>
            <svg width={13} height={13} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2} strokeLinecap="round" strokeLinejoin="round">
              <line x1="19" y1="12" x2="5" y2="12"/><polyline points="12 19 5 12 12 5"/>
            </svg>
            Back to Projects
          </button>
          <h1 style={{ fontFamily: "'Barlow Condensed', sans-serif", fontSize: "clamp(22px, 4vw, 30px)", fontWeight: 900, textTransform: "uppercase", letterSpacing: "0.02em", lineHeight: 1 }}>
            Daily Progress Report
          </h1>
          <p style={{ fontSize: 13, color: T.textMuted, marginTop: 5 }}>
            File a new DPR for site activities, workforce, and conditions
          </p>
        </div>

        <form onSubmit={handleSubmit} noValidate>
          <div style={{
            display: "grid", gridTemplateColumns: "1fr", gap: 0,
            opacity: mounted ? 1 : 0, transition: "opacity 0.4s ease 0.1s",
          }}>
            {/* Section: Project Info */}
            <Section title="Project Information" icon="layers" T={T}>
              <div style={{ display: "grid", gridTemplateColumns: "repeat(auto-fit, minmax(240px, 1fr))", gap: "0 20px" }}>
                <FormField label="Select Project" error={errors.projectId} required>
                  <div style={{ position: "relative" }}>
                    <select value={form.projectId} onChange={set("projectId")} style={{
                      ...inputStyle(!!errors.projectId), paddingRight: 36,
                      backgroundImage: `url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='8' viewBox='0 0 12 8'%3E%3Cpath d='M1 1l5 5 5-5' stroke='%238694aa' stroke-width='1.5' fill='none' stroke-linecap='round'/%3E%3C/svg%3E")`,
                      backgroundRepeat: "no-repeat", backgroundPosition: "right 12px center",
                    }} {...focusHandlers("projectId")}>
                      <option value="">— Select a project —</option>
                      {PROJECTS.map(p => <option key={p.id} value={p.id}>{p.id} — {p.name}</option>)}
                    </select>
                  </div>
                </FormField>

                {selectedProject && (
                  <div style={{ marginBottom: 18 }}>
                    <div style={{ fontFamily: "'Barlow Condensed', sans-serif", fontSize: 11, fontWeight: 700, letterSpacing: "0.12em", textTransform: "uppercase", color: T.textSub, marginBottom: 5 }}>Project Preview</div>
                    <div style={{ padding: "10px 14px", background: T.card, border: `1px solid ${T.border}`, borderRadius: 5, display: "flex", justifyContent: "space-between", alignItems: "center", gap: 8 }}>
                      <div>
                        <div style={{ fontFamily: "'Barlow Condensed', sans-serif", fontSize: 14, fontWeight: 700 }}>{selectedProject.name}</div>
                        <div style={{ fontSize: 11.5, color: T.textMuted, marginTop: 2 }}>{selectedProject.location}</div>
                      </div>
                      <Badge status={selectedProject.status} />
                    </div>
                  </div>
                )}
              </div>
            </Section>

            {/* Section: Site Conditions */}
            <Section title="Site Conditions" icon="cloud" T={T}>
              <div style={{ display: "grid", gridTemplateColumns: "repeat(auto-fit, minmax(200px, 1fr))", gap: "0 20px" }}>
                <FormField label="Report Date" error={errors.date} required>
                  <input type="date" value={form.date} onChange={set("date")}
                    max={todayISO()} style={inputStyle(!!errors.date)} {...focusHandlers("date")} />
                </FormField>
                <FormField label="Weather Conditions" error={errors.weather} required>
                  <select value={form.weather} onChange={set("weather")} style={{
                    ...inputStyle(!!errors.weather), paddingRight: 36,
                    backgroundImage: `url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='8' viewBox='0 0 12 8'%3E%3Cpath d='M1 1l5 5 5-5' stroke='%238694aa' stroke-width='1.5' fill='none' stroke-linecap='round'/%3E%3C/svg%3E")`,
                    backgroundRepeat: "no-repeat", backgroundPosition: "right 12px center",
                  }} {...focusHandlers("weather")}>
                    {WEATHER_OPTIONS.map(w => <option key={w.value} value={w.value}>{w.label}</option>)}
                  </select>
                </FormField>
              </div>
            </Section>

            {/* Section: Workforce */}
            <Section title="Workforce Details" icon="users" T={T}>
              <div style={{ display: "grid", gridTemplateColumns: "repeat(auto-fit, minmax(200px, 1fr))", gap: "0 20px" }}>
                <FormField label="Worker Count" error={errors.workerCount} required>
                  <input type="number" value={form.workerCount} min={1} max={5000} placeholder="e.g. 45"
                    onChange={set("workerCount")} style={inputStyle(!!errors.workerCount)} {...focusHandlers("workerCount")} />
                </FormField>
                <FormField label="Site Supervisor Name">
                  <input type="text" value={form.supervisorName} placeholder="e.g. Ramesh Rao"
                    onChange={set("supervisorName")} style={inputStyle(false)} {...focusHandlers("supervisorName")} />
                </FormField>
              </div>
            </Section>

            {/* Section: Work Description */}
            <Section title="Work Performed" icon="clipboard" T={T}>
              <FormField label="Work Description" error={errors.workDescription} required>
                <div style={{ position: "relative" }}>
                  <textarea value={form.workDescription} rows={5} placeholder="Describe all work activities carried out today on site (minimum 20 characters). Include details about tasks completed, areas worked, materials used, and any progress milestones..."
                    onChange={set("workDescription")} style={{
                      ...inputStyle(!!errors.workDescription),
                      resize: "vertical", minHeight: 110, lineHeight: 1.65, padding: "10px 14px",
                    }} {...focusHandlers("workDescription")} />
                  <span style={{
                    position: "absolute", bottom: 8, right: 12, fontSize: 10.5,
                    color: form.workDescription.length < 20 ? T.red : T.textMuted, fontFamily: "'Barlow Condensed', sans-serif",
                  }}>
                    {form.workDescription.length} / min 20
                  </span>
                </div>
              </FormField>
              <FormField label="Remarks / Issues">
                <textarea value={form.remarks} rows={3} placeholder="Any issues, delays, safety incidents, or additional remarks..."
                  onChange={set("remarks")} style={{ ...inputStyle(false), resize: "vertical", minHeight: 72, lineHeight: 1.65, padding: "10px 14px" }}
                  {...focusHandlers("remarks")} />
              </FormField>
            </Section>

            {/* Section: Photo Upload */}
            <Section title="Site Photos" icon="camera" T={T}>
              <div style={{ marginBottom: 16 }}>
                <div style={{ fontFamily: "'Barlow Condensed', sans-serif", fontSize: 11, fontWeight: 700, letterSpacing: "0.12em", textTransform: "uppercase", color: T.textSub, marginBottom: 8 }}>
                  Photos <span style={{ color: T.textMuted, fontWeight: 400 }}>({photos.length}/3 selected — optional)</span>
                </div>

                {photos.length < 3 && (
                  <div
                    onClick={() => fileRef.current?.click()}
                    style={{
                      border: `2px dashed ${T.border}`, borderRadius: 6, padding: "28px 20px",
                      textAlign: "center", cursor: "pointer", marginBottom: 12,
                      background: T.card, transition: "border-color 0.2s, background 0.2s",
                    }}
                    onMouseEnter={e => { e.currentTarget.style.borderColor = T.amber; e.currentTarget.style.background = T.amberGlow; }}
                    onMouseLeave={e => { e.currentTarget.style.borderColor = T.border; e.currentTarget.style.background = T.card; }}
                  >
                    <div style={{ display: "flex", flexDirection: "column", alignItems: "center", gap: 8, pointerEvents: "none" }}>
                      <svg width={32} height={32} viewBox="0 0 24 24" fill="none" stroke={T.textMuted} strokeWidth={1.5} strokeLinecap="round" strokeLinejoin="round">
                        <rect x="3" y="3" width="18" height="18" rx="2" ry="2"/><circle cx="8.5" cy="8.5" r="1.5"/><polyline points="21 15 16 10 5 21"/>
                      </svg>
                      <div>
                        <p style={{ fontFamily: "'Barlow Condensed', sans-serif", fontSize: 14, fontWeight: 700, color: T.textSub, letterSpacing: "0.04em" }}>
                          Click to upload site photos
                        </p>
                        <p style={{ fontSize: 11.5, color: T.textMuted, marginTop: 3 }}>PNG, JPG, WEBP · Max 3 images</p>
                      </div>
                    </div>
                    <input ref={fileRef} type="file" accept="image/*" multiple onChange={handlePhotos} style={{ display: "none" }} />
                  </div>
                )}

                {photos.length > 0 && (
                  <div style={{ display: "flex", gap: 10, flexWrap: "wrap" }}>
                    {photos.map(ph => (
                      <div key={ph.id} style={{
                        position: "relative", width: 100, height: 100, borderRadius: 6,
                        overflow: "hidden", border: `1px solid ${T.border}`, flexShrink: 0,
                      }}>
                        <img src={ph.url} alt={ph.name} style={{ width: "100%", height: "100%", objectFit: "cover" }} />
                        <button onClick={() => removePhoto(ph.id)} style={{
                          position: "absolute", top: 4, right: 4, width: 20, height: 20,
                          background: "rgba(0,0,0,0.7)", border: "none", borderRadius: "50%",
                          color: "#fff", cursor: "pointer", display: "flex", alignItems: "center", justifyContent: "center",
                        }}>
                          <svg width={10} height={10} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2.5} strokeLinecap="round" strokeLinejoin="round">
                            <line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/>
                          </svg>
                        </button>
                        <div style={{
                          position: "absolute", bottom: 0, left: 0, right: 0,
                          background: "rgba(0,0,0,0.6)", padding: "3px 5px",
                          fontSize: 9, color: "#ddd", fontFamily: "'Barlow Condensed', sans-serif",
                          whiteSpace: "nowrap", overflow: "hidden", textOverflow: "ellipsis",
                        }}>
                          {(ph.size / 1024).toFixed(0)} KB
                        </div>
                      </div>
                    ))}
                    {photos.length < 3 && (
                      <div onClick={() => fileRef.current?.click()} style={{
                        width: 100, height: 100, border: `2px dashed ${T.border}`, borderRadius: 6,
                        display: "flex", alignItems: "center", justifyContent: "center",
                        cursor: "pointer", color: T.textMuted, flexShrink: 0, background: T.card,
                        transition: "border-color 0.2s",
                      }}
                        onMouseEnter={e => e.currentTarget.style.borderColor = T.amber}
                        onMouseLeave={e => e.currentTarget.style.borderColor = T.border}
                      >
                        <svg width={20} height={20} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2} strokeLinecap="round" strokeLinejoin="round">
                          <line x1="12" y1="5" x2="12" y2="19"/><line x1="5" y1="12" x2="19" y2="12"/>
                        </svg>
                      </div>
                    )}
                  </div>
                )}
              </div>
            </Section>

            {/* Submit */}
            <div style={{
              display: "flex", gap: 10, flexWrap: "wrap",
              paddingTop: 8,
            }}>
              <button type="button" onClick={() => navigate("projects")} style={{
                flex: "0 1 auto", padding: "11px 20px",
                background: "none", border: `1px solid ${T.border}`, borderRadius: 5,
                color: T.textSub, fontFamily: "'Barlow Condensed', sans-serif",
                fontSize: 13, fontWeight: 700, letterSpacing: "0.1em", textTransform: "uppercase", cursor: "pointer",
                display: "flex", alignItems: "center", gap: 6,
              }}>
                <svg width={14} height={14} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2} strokeLinecap="round" strokeLinejoin="round">
                  <line x1="19" y1="12" x2="5" y2="12"/><polyline points="12 19 5 12 12 5"/>
                </svg>
                Cancel
              </button>
              <button type="button" onClick={handleReset} style={{
                flex: "0 1 auto", padding: "11px 18px",
                background: "none", border: `1px solid ${T.border}`, borderRadius: 5,
                color: T.textSub, fontFamily: "'Barlow Condensed', sans-serif",
                fontSize: 13, fontWeight: 700, letterSpacing: "0.1em", textTransform: "uppercase", cursor: "pointer",
              }}>
                Reset Form
              </button>
              <button type="submit" disabled={submitting} style={{
                flex: "1 1 200px", padding: "11px 24px",
                background: submitting ? T.amberGlow : T.amber,
                border: submitting ? `1px solid ${T.amberBorder}` : "none",
                borderRadius: 5, color: submitting ? T.amber : "#000",
                fontFamily: "'Barlow Condensed', sans-serif", fontSize: 14, fontWeight: 800,
                letterSpacing: "0.1em", textTransform: "uppercase", cursor: submitting ? "not-allowed" : "pointer",
                display: "flex", alignItems: "center", justifyContent: "center", gap: 8,
                transition: "all 0.2s",
              }}>
                {submitting ? (
                  <><div style={{ width: 16, height: 16, border: `2px solid ${T.amberBorder}`, borderTopColor: T.amber, borderRadius: "50%", animation: "spin 0.6s linear infinite" }} />Submitting DPR...</>
                ) : (
                  <>Submit Daily Progress Report <svg width={15} height={15} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2.5} strokeLinecap="round" strokeLinejoin="round"><path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"/><path d="M22 4L12 14.01l-3-3"/></svg></>
                )}
              </button>
            </div>
          </div>
        </form>
      </div>
    </div>
  );
}

function Section({ title, icon, T, children }) {
  const icons = {
    layers: <><polygon points="12 2 2 7 12 12 22 7 12 2"/><polyline points="2 17 12 22 22 17"/><polyline points="2 12 12 17 22 12"/></>,
    cloud: <><path d="M18 10h-1.26A8 8 0 1 0 9 20h9a5 5 0 0 0 0-10z"/></>,
    users: <><path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M23 21v-2a4 4 0 0 0-3-3.87"/><path d="M16 3.13a4 4 0 0 1 0 7.75"/></>,
    clipboard: <><path d="M16 4h2a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2H6a2 2 0 0 1-2-2V6a2 2 0 0 1 2-2h2"/><rect x="8" y="2" width="8" height="4" rx="1" ry="1"/></>,
    camera: <><path d="M23 19a2 2 0 0 1-2 2H3a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h4l2-3h6l2 3h4a2 2 0 0 1 2 2z"/><circle cx="12" cy="13" r="4"/></>,
  };

  return (
    <div style={{ marginBottom: 4 }}>
      <div style={{
        display: "flex", alignItems: "center", gap: 8, padding: "12px 0 10px",
        borderBottom: `1px solid ${T.border}`, marginBottom: 20,
      }}>
        <div style={{
          width: 28, height: 28, background: T.amberGlow, border: `1px solid ${T.amberBorder}`,
          borderRadius: 4, display: "flex", alignItems: "center", justifyContent: "center", color: T.amber, flexShrink: 0,
        }}>
          <svg width={14} height={14} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={2} strokeLinecap="round" strokeLinejoin="round">
            {icons[icon]}
          </svg>
        </div>
        <h3 style={{ fontFamily: "'Barlow Condensed', sans-serif", fontSize: 14, fontWeight: 800, letterSpacing: "0.1em", textTransform: "uppercase", color: T.textSub }}>
          {title}
        </h3>
      </div>
      {children}
    </div>
  );
}
```
