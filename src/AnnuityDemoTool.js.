// ------------------------------------------------------
//  Annuity Demo Tool – FULL SOURCE (CLEANED & FIXED + COMPONENT BOOTSTRAP)
// ------------------------------------------------------

import React, { useState, useEffect, useRef } from "react";
import * as XLSX from "xlsx";
import { motion, AnimatePresence } from "framer-motion";
import { Button } from "@/components/ui/button";
import { Card, CardContent } from "@/components/ui/card";
import { Input } from "@/components/ui/input";
import { Table, TableBody, TableCell, TableHeader, TableRow } from "@/components/ui/table";
import { DndProvider, useDrag, useDrop } from "react-dnd";
import { HTML5Backend } from "react-dnd-html5-backend";
import {
  ResponsiveContainer,
  PieChart,
  Pie,
  Cell,
  Legend,
  BarChart,
  Bar,
  XAxis,
  YAxis,
  Tooltip as RechartTooltip,
  LineChart,
  Line,
  CartesianGrid
} from "recharts";

// --------------------------------
// TabButtons component definition
// --------------------------------
function TabButtons({ activeTab, setActiveTab }) {
  const tabs = ["Advisor", "Wholesaler", "Reports"];
  return (
    <div className="flex gap-2">
      {tabs.map((tab) => (
        <Button
          key={tab}
          size="sm"
          className={`text-sm px-4 py-2 rounded font-semibold border ${
            activeTab === tab
              ? 'bg-blue-600 text-white'
              : 'bg-transparent text-gray-300 border-gray-500 hover:bg-blue-700 hover:text-white'
          }`}
          onClick={() => setActiveTab(tab)}
        >
          {tab} View
        </Button>
      ))}
    </div>
  );
}

const introMessages = [
  "Managing a book of annuities can feel daunting—paralysis by analysis waiting around every corner.",
  "Contracts age, markets shift, and clients’ goals evolve—making what once fit no longer enough.",
  "Our tool cuts through the complexity, pinpointing which policies to review first and with whom.",
  "Upload your annuity book and get a clear, prioritized roadmap in moments."
];

export function FlashIntroMain({ onStart }) {
  const [step, setStep] = useState(0);
  useEffect(() => {
    if (step >= introMessages.length) {
      const timer = setTimeout(onStart, 500);
      return () => clearTimeout(timer);
    }
    const timer = setTimeout(() => setStep((s) => s + 1), 3300);
    return () => clearTimeout(timer);
  }, [step, onStart]);

  return (
    <div className="min-h-screen bg-gradient-to-b from-[#192234] to-[#092058] text-white flex flex-col items-center justify-center p-6 p-12 text-center">
     <motion.h1
  initial={{ opacity: 0, y: -20 }}
  animate={{ opacity: 1, y: 0 }}
  transition={{ duration: 1 }}
  className="text-6xl font-extrabold tracking-wide mb-10 text-white text-center drop-shadow-md"
>
  Clarity Where It Counts Most
</motion.h1>

      <AnimatePresence mode="wait">
        <motion.div
          key={step}
          initial={{ opacity: 0, y: 20 }}
          animate={{ opacity: 1, y: 0 }}
          exit={{ opacity: 0, y: -20 }}
          transition={{ duration: 0.8 }}
          className="text-center text-2xl max-w-xl"
        >
          {introMessages[step]}
        </motion.div>
      </AnimatePresence>
      <div className="mt-6">
        {step < introMessages.length ? (
          <Button className="bg-[#2B3556] text-white border-white hover:bg-[#1a1f38]" onClick={onStart}>Skip</Button>
        ) : (
          <Button onClick={onStart}>Start</Button>
        )}
      </div>
    </div>
  );
}

export function Spinner() {
  return (
    <div className="fixed inset-0 bg-black bg-opacity-60 flex items-center justify-center z-50">
      <div className="flex flex-col items-center">
        <div className="w-16 h-16 border-t-4 border-b-4 border-blue-500 rounded-full animate-spin"></div>
        <p className="mt-4 text-lg font-semibold text-white animate-pulse">Loading...</p>
      </div>
    </div>
  );
}


/* =====================================================
   SMALL DRAG‑N‑DROP CARD COMPONENT (for dashboard layout)
   =====================================================*/
const ItemType = { CARD: "card" };
function DraggableCard({ id, index, moveCard, children }) {
  const ref = useRef(null);
  const [{ isDragging }, drag] = useDrag({
    type: ItemType.CARD,
    item: { id, index },
    collect: (monitor) => ({ isDragging: monitor.isDragging() })
  });
  const [, drop] = useDrop({
    accept: ItemType.CARD,
    hover(item, monitor) {
      if (!ref.current) return;
      const dragIndex = item.index;
      const hoverIndex = index;
      if (dragIndex === hoverIndex) return;
      const { top, bottom } = ref.current.getBoundingClientRect();
      const hoverMiddleY = (bottom - top) / 2;
      const clientY = monitor.getClientOffset().y - top;
      if (
        (dragIndex < hoverIndex && clientY < hoverMiddleY) ||
        (dragIndex > hoverIndex && clientY > hoverMiddleY)
      )
        return;
      moveCard(dragIndex, hoverIndex);
      item.index = hoverIndex;
    }
  });
  drag(drop(ref));
  return (
    <div
      ref={ref}
      style={{ opacity: isDragging ? 0.5 : 1, cursor: "move", marginBottom: 12 }}
    >
      {children}
    </div>
  );
}

/* =====================================================
   CONSTANTS & HELPER FUNCTIONS
   =====================================================*/
const nariaCorePayoutRates = [
  { minAge: 80, rate: 0.0665 },
  { minAge: 75, rate: 0.065 },
  { minAge: 70, rate: 0.063 },
  { minAge: 65, rate: 0.061 },
  { minAge: 60, rate: 0.0475 },
  { minAge: 55, rate: 0.0355 },
  { minAge: 45, rate: 0.0355 }
];
const getNariaPayoutRate = (age) => {
  for (const row of nariaCorePayoutRates) {
    if (age >= row.minAge) return row.rate;
  }
  return 0.0355;
};

function getNariaCoreProjection(
  startVal,
  years,
  incomeStart,
  payoutRate = 0.05,
  retSeries = [],
  rollupRate = 0.08,
  initialBenefitBase
) {
  const startYear = new Date().getFullYear();
  let benefitBase =
    initialBenefitBase !== undefined ? initialBenefitBase : startVal;
  let contractValue = startVal;
  const maxRollupYears = 10;
  let income = 0;
  const pts = [];

  for (let i = 0; i <= years; i++) {
    const year = startYear + i;
    const rawRet = retSeries[i] !== undefined ? retSeries[i] : 0;
    const ret = Math.max(-0.25, Math.min(rawRet, 0.15));
    let grow = ret;
    let rollupUsed = false;
    if (i < incomeStart) {
      if (ret < rollupRate) {
        grow = rollupRate;
        rollupUsed = true;
      }
    }

    if (i < incomeStart && i < maxRollupYears) {
      contractValue *= 1 + grow;
      benefitBase *= 1 + grow;
    } else {
      if (i === incomeStart) {
        income = benefitBase * payoutRate;
      }
      contractValue = Math.max(0, contractValue - income);
      benefitBase = Math.max(0, benefitBase - income);
    }

    pts.push({
      year,
      value: contractValue,
      income: i >= incomeStart ? income : 0,
      benefitBase,
      avgReturn: ret,
      appliedReturn: grow,
      rollupUsed
    });
  }

  pts.sort((a, b) => a.year - b.year);
  return pts;
}

/* =====================================================
   MAIN APP COMPONENT
   =====================================================*/
export default function AnnuityDemoTool() {
  /* ---- top‑level state ---- */
  const [showIntro, setShowIntro] = useState(true);
  const [uploaded, setUploaded] = useState(false);
  const [data, setData] = useState([]);
  const [activeTab, setActiveTab] = useState("Advisor");
  const [selectedRows, setSelectedRows] = useState([]);
  const [pipelineRows, setPipelineRows] = useState([]);
  const [reportRows, setReportRows] = useState([]);
  const [sections, setSections] = useState(["statusChart", "valueChart", "subCategories"]);
  const [comparisonRequests, setComparisonRequests] = useState([]);
  const [requestScenarios, setRequestScenarios] = useState({});

  /* ---- projection controls ---- */
  const [showProjection, setShowProjection] = useState(false);
  const [expandedProjection, setExpandedProjection] = useState(false);
  const [allocAgg, setAllocAgg] = useState(0.6);
  const [incomeStart, setIncomeStart] = useState(1);
  const [projectionData, setProjectionData] = useState([]);
  const [selectedScenario, setSelectedScenario] = useState(null);
  const [comparisonData, setComparisonData] = useState([]);
  const [projMode, setProjMode] = useState("montecarlo");
  const sumIncomeYTD = (series, year) =>
  series.reduce((acc, pt) => acc + ((pt?.year <= year && pt?.income) || 0), 0);
  const nationwideOptions = [
    { id: "naria_core", label: "NARIA + CORE" },
    { id: "naria_accel", label: "NARIA + ACCEL" },
    { id: "monument", label: "Monument Advisor" }
  ];

  /* ---------------- FILE UPLOAD ---------------- */
  function Spinner() {
  return (
    <div className="fixed inset-0 bg-black bg-opacity-60 flex items-center justify-center z-50">
      <div className="flex flex-col items-center">
        <div className="w-16 h-16 border-t-4 border-b-4 border-blue-500 rounded-full animate-spin"></div>
        <p className="mt-4 text-lg font-semibold text-white animate-pulse">Loading...</p>
      </div>
    </div>
  );
}

const handleFileUpload = (e) => {
    const file = e.target.files ? e.target.files[0] : null;
    if (!file) return;
    const reader = new FileReader();
    reader.onload = (evt) => {
      const wb = XLSX.read(evt.target.result, { type: "array" });
      const ws = wb.Sheets[wb.SheetNames[0]];
      setData(XLSX.utils.sheet_to_json(ws, { defval: "" }));
      setUploaded(true);
    };
    reader.readAsArrayBuffer(file);
  };

  const resetAll = () => {
    setUploaded(false);
    setData([]);
    setSelectedRows([]);
    setPipelineRows([]);
    setReportRows([]);
    setActiveTab("Advisor");
    setSections(["statusChart", "valueChart", "subCategories"]);
  };

  /* ---------------- PROJECTION LOGIC ---------------- */
  useEffect(() => {
    if (!showProjection) return;
    const years = 30;
    const startVal = selectedRows.reduce((sum, name) => {
      const rec = data.find((r) => r["Client Name"] === name);
      return sum + (rec ? parseFloat(rec["Contract Value"]) || 0 : 0);
    }, 0);
    const avgPayout =
      selectedRows.reduce((sum, name) => {
        const rec = data.find((r) => r["Client Name"] === name);
        return sum + (rec ? parseFloat(String(rec["Payout Rate"]).replace(/%/g, "")) || 0 : 0);
      }, 0) /
        (selectedRows.length || 1) /
        100;
    const rollupRate =
      selectedRows.reduce((sum, name) => {
        const rec = data.find((r) => r["Client Name"] === name);
        return sum + (rec ? parseFloat(String(rec["Roll Up Rate"]).replace(/%/g, "")) || 0 : 0);
      }, 0) /
        (selectedRows.length || 1) /
        100;

    const mu = 0.07,
      sigma = 0.15;
    const retSeries = projMode === "fixed"
      ? Array(years + 1).fill(rollupRate)
      : Array.from({ length: years + 1 }, () => {
          const r = mu + sigma * (Math.random() * 2 - 1);
          return Math.max(-0.25, Math.min(r, 0.15));
        });

    const clientProj = getNariaCoreProjection(
      startVal,
      years,
      incomeStart,
      avgPayout,
      retSeries,
      rollupRate,
      startVal
    );
    setProjectionData(clientProj);

    if (selectedScenario === "naria_core") {
      const rec = data.find((r) => r["Client Name"] === selectedRows[0]);
      if (rec) {
        const age = parseInt(rec["Client Age"]) || 65;
        const payoutRate = getNariaPayoutRate(age + incomeStart);
        setComparisonData(
          getNariaCoreProjection(
            parseFloat(rec["Contract Value"]) || 0,
            years,
            incomeStart,
            payoutRate,
            retSeries,
            0.08,
            parseFloat(rec["Contract Value"]) || 0
          )
        );
      }
    } else {
      setComparisonData([]);
    }
  }, [showProjection, projMode, incomeStart, selectedRows, data, selectedScenario]);

  /* -------------- misc helpers -------------- */
  const formatPct = (v) => `${(parseFloat(String(v).replace(/%/g, "")) || 0).toFixed(2)}%`;
  const statusColors = {
    "In Sync": "#6ECEB2",
    "Consider Review": "#092058",
    "More Info Needed": "#EDE8D0"
  };
  const groupByStatus = (recs) =>
    recs.reduce((acc, r) => {
      const k = r.Status || "Unknown";
      (acc[k] = acc[k] || []).push(r);
      return acc;
    }, {});

  const opportunitySnapshot = (arr) => {
    const total = arr.length;
    const lowPayout = arr.filter((d) => parseFloat(d["Payout Rate"]) < 4).length;
    const shortTTI = arr.filter((d) => parseFloat(d["Time to Income"]) <= 2).length;
    const droppingValue = arr.filter((d) => parseFloat(d["Change in Value"]) < 0).length;
    return { total, lowPayout, shortTTI, droppingValue };
  };

  /* ------------ table & section render helpers ------------ */
  const toggleRow = (name) =>
    setSelectedRows((prev) =>
      prev.includes(name) ? prev.filter((n) => n !== name) : [...prev, name]
    );
  const togglePipeline = (name) =>
    setPipelineRows((prev) =>
      prev.includes(name) ? prev.filter((n) => n !== name) : [...prev, name]
    );
  const toggleReport = (name) =>
    setReportRows((prev) =>
      prev.includes(name) ? prev.filter((n) => n !== name) : [...prev, name]
    );

  const renderTable = (recs) => {
    if (!recs.length) return null;
    const cols = Object.keys(recs[0]);
    const hasSel = selectedRows.length > 0;
    const isWh = activeTab === "Wholesaler";
    const isAdv = activeTab === "Advisor";
    const allSel = recs.every((r) => selectedRows.includes(r["Client Name"]));
    return (
      <div className="overflow-auto w-full">
        <Table>
          <TableHeader>
            <TableRow>
              <TableCell>
                <input
                  type="checkbox"
                  checked={allSel}
                  onChange={(e) =>
                    setSelectedRows(
                      e.target.checked ? recs.map((r) => r["Client Name"]) : []
                    )
                  }
                />
              </TableCell>
              <TableCell className="font-bold text-[#1F74DB]">Client Name</TableCell>
              {cols
                .filter((c) => c !== "Client Name")
                .map((c) => (
                  <TableCell key={c} className="font-bold text-[#1F74DB]">
                    {c}
                  </TableCell>
                ))}
              {isAdv && hasSel && <TableCell className="font-bold text-[#1F74DB]">Request Comparison</TableCell>}
              {isAdv && hasSel && <TableCell className="font-bold text-[#1F74DB]">Scenario</TableCell>}
              {isWh && hasSel && <TableCell className="font-bold text-[#1F74DB]">Pipeline</TableCell>}
              {isWh && hasSel && <TableCell className="font-bold text-[#1F74DB]">Scenario</TableCell>}
              {isWh && hasSel &&<TableCell className="font-bold text-[#1F74DB]">Action</TableCell>}
            </TableRow>
          </TableHeader>
          <TableBody>
            {recs.map((r, i) => (
              <TableRow
                key={i}
                className={`text-white ${selectedRows.includes(r["Client Name"]) ? "bg-[#1a1f38]" : "hover:bg-[#0f1e3e]"}`}
              >
                {/* Checkbox */}
                <TableCell>
                  <input
                    type="checkbox"
                    checked={selectedRows.includes(r["Client Name"]) }
                    onChange={() => toggleRow(r["Client Name"]) }
                  />
                </TableCell>
                {/* Client Name */}
                <TableCell>{r["Client Name"]}</TableCell>
                {/* Data columns */}
                {cols.filter(c => c !== "Client Name").map((c, j) => {
                  let d = r[c];
                  if (["Roll Up Rate", "Payout Rate"].includes(c)) d = formatPct(d);
                  if (["Contract Value", "Benefit Base"].includes(c)) d = `$${(parseFloat(d) || 0).toLocaleString()}`;
                  const style = c === "Status" && selectedRows.includes(r["Client Name"]) ? { backgroundColor: statusColors[r.Status], color: "#fff" } : {};
                  return (
                    <TableCell key={j} style={style}>
                      {d}
                    </TableCell>
                  );
                })}
                {/* Advisor View: Request Comparison */}
                {isAdv && (
                  <TableCell>
                    {hasSel && (
                      <Button
                        size="sm"
                        onClick={() => {
                          toggleReport(r["Client Name"]);
                          if (!comparisonRequests.includes(r["Client Name"])) {
                            setComparisonRequests([...comparisonRequests, r["Client Name"]]);
                          }
                        }}
                        disabled={comparisonRequests.includes(r["Client Name"]) }
                      >
                        {comparisonRequests.includes(r["Client Name"]) ? "✓ Requested" : "Request Comparison"}
                      </Button>
                    )}
                  </TableCell>
                )}
                {/* Advisor View: Scenario */}
                {isAdv && (
                  <TableCell>
                    {comparisonRequests.includes(r["Client Name"]) && (
                      <select
                        className="bg-gray-700 border border-white text-white px-2 py-1 rounded w-full"
                        value={requestScenarios[r["Client Name"]] || ""}
                        onChange={e => setRequestScenarios(prev => ({ ...prev, [r["Client Name"]]: e.target.value }))}
                      >
                        <option value="">Select Scenario</option>
                        {nationwideOptions.map(o => (
                          <option key={o.id} value={o.label}>{o.label}</option>
                        ))}
                      </select>
                    )}
                  </TableCell>
                )}
                {/* Wholesaler View: Pipeline */}
                {isWh && (
                  <TableCell>
                    {selectedRows.includes(r["Client Name"]) && (
                      <Button
  size="sm"
  className={`text-sm px-2 py-1 rounded ${
    pipelineRows.includes(r["Client Name"]) ? 'bg-green-600 text-white' : 'bg-blue-600 text-white'
  }`}
  onClick={() => {
    const added = !pipelineRows.includes(r["Client Name"]);
    togglePipeline(r["Client Name"]);
    if (added) alert('Success! Added to pipeline.');
  }}
>
  {pipelineRows.includes(r["Client Name"]) ? "✓ Added" : "Add to Pipeline"}
</Button>
                    )}
                  </TableCell>
                )}
                {/* Wholesaler View: Scenario */}
                {isWh && (
                  <TableCell>
                    {selectedRows.includes(r["Client Name"]) && (
                      <select className="bg-gray-700 border border-white text-white px-2 py-1 rounded w-full">
                        <option>Monument Advisor</option>
                        <option>NARIA + CORE</option>
                        <option>NARIA + ACCEL</option>
                      </select>
                    )}
                  </TableCell>
                )}
                {/* Wholesaler View: What If */}
                {isWh && (
                  <TableCell>
                    {selectedRows.includes(r["Client Name"]) && (
                      <Button size="sm" onClick={() => setShowProjection(true)}>What If</Button>
                    )}
                  </TableCell>
                )}
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </div>
    );
  };

  const renderStatusLegend = (props) => {
  const { payload } = props;
  return (
    <ul className="flex justify-center gap-4 text-sm text-gray-300 font-medium">
      {payload.map((entry, index) => (
        <li key={`item-${index}`} className="flex items-center gap-2">
          <span
            className="inline-block w-3 h-3 rounded-full"
            style={{ backgroundColor: entry.color }}
          ></span>
          <span>{entry.value}</span>
        </li>
      ))}
    </ul>
  );
};
const renderSection = (key) => {
    const grouped = groupByStatus(data);
    switch (key) {
      case "statusChart":
        return (
          <DraggableCard
            id="statusChart"
            index={sections.indexOf("statusChart")}
            moveCard={(from, to) => {
              const s = [...sections];
              const m = s.splice(from, 1)[0];
              s.splice(to, 0, m);
              setSections(s);
            }}
          >
            <div className="p-4 mb-4 w-[calc(100%-0.5rem)]">
              <CardContent>
                <h3 className="font-bold mb-2 text-[#1F74DB]">Status Breakdown</h3>
                <div
                  className="overflow-hidden rounded p-2"
                  style={{ resize: "both", minWidth: "300px", minHeight: "250px" }}
                >
                  <ResponsiveContainer width="100%" height={expandedProjection ? 500 : 300}>
                    <PieChart>
                      <Pie
                        data={Object.entries(grouped).map(([n, a]) => ({ name: n, value: a.length }))}
                        dataKey="value"
                        innerRadius={40}
                        outerRadius={70}
                        paddingAngle={2}
                      >
                        {Object.keys(grouped).map((n, i) => (
                          <Cell key={i} fill={statusColors[n] || "#ccc"} />
                        ))}
                      </Pie>
                      <Legend content={renderStatusLegend} />
                      <RechartTooltip
                        content={({ active, payload }) => {
                          if (!active || !payload || !payload[0]) return null;
                          const name = payload[0].payload.name;
                          const arr = grouped[name] || [];
                          const avgRoll =
                            arr.reduce((sum, r) => sum + (parseFloat(r["Roll Up Rate"]) || 0), 0) /
                            (arr.length || 1);
                          const avgPayout =
                            arr.reduce((sum, r) => sum + (parseFloat(r["Payout Rate"]) || 0), 0) /
                            (arr.length || 1);
                          const ttiVals = arr.map(r => {
  // handle possible variants of the column name
  const raw = r["Time to Income"] ?? r["Time To Income"] ?? r["TTI"] ?? 0;
  // strip non-numeric characters and parse
  const num = parseFloat(String(raw).replace(/[^0-9.\-]+/g, "")) || 0;
  return num;
});
const avgTTI = ttiVals.length
  ? (ttiVals.reduce((sum, v) => sum + v, 0) / ttiVals.length).toFixed(1)
  : "—";
                          return (
                            <div className="p-2 bg-gray-100 border border-gray-400 rounded shadow-lg text-gray-800 text-sm">
                              <strong>{name}</strong>
                              <br />
                              Avg Roll Up Rate: {avgRoll.toFixed(2)}%
                              <br />
                              Avg Payout Rate: {avgPayout.toFixed(2)}%
                              <br />
                              Avg TTI: {avgTTI} yrs
                            </div>
                          );
                        }}
                      />
                    </PieChart>
                  </ResponsiveContainer>
                </div>
              </CardContent>
            </div>
          </DraggableCard>
        );
      case "valueChart":
        return (
          <DraggableCard
            id="valueChart"
            index={sections.indexOf("valueChart")}
            moveCard={(from, to) => {
              const s = [...sections];
              const m = s.splice(from, 1)[0];
              s.splice(to, 0, m);
              setSections(s);
            }}
          >
            <div className="p-4 mb-4 w-full">
              <CardContent>
                <h3 className="font-bold mb-2 text-[#1F74DB]">Contract Values</h3>
                <div
                  className="resize overflow-auto rounded p-2"
                  style={{ width: "100%", minWidth: "300px", minHeight: "250px" }}
                >
                  <ResponsiveContainer width="100%" height={expandedProjection ? 500 : 300}>
                    <BarChart data={data} margin={{ top: 5, right: 20, left: 80, bottom: 5 }}>
                      <YAxis tickFormatter={(v) => `$${v.toLocaleString()}`} />
                      <RechartTooltip
                        content={({ active, payload }) =>
                          active && payload && payload[0] ? (
                            <div className="p-2 bg-gray-100 border border-gray-400 rounded shadow-lg text-gray-800 text-sm">
                              <strong>{payload[0].payload["Client Name"]}</strong>
                              <br />
                              Carrier: {payload[0].payload.Carrier}
                              <br />
                              Product: {payload[0].payload.Product}
                              <br />
                              Value: $
                              {Number(payload[0].payload["Contract Value"]).toLocaleString()}
                              <br />
                              Status: {payload[0].payload.Status}
                            </div>
                          ) : null
                        }
                      />
                      <Bar dataKey="Contract Value" fill="#C5C9D4">
                        {data.map((_, i) => (
                          <Cell key={i} />
                        ))}
                      </Bar>
                    </BarChart>
                  </ResponsiveContainer>
                </div>
              </CardContent>
            </div>
          </DraggableCard>
        );
      case "subCategories":
        return (
          <div className="w-full">
            {Object.entries(grouped).map(([status, recs]) => (
              <Card
                key={status}
                className="p-4 mb-4 bg-transparent text-white border border-white hover:shadow-xl transition-shadow duration-300 w-full backdrop-blur-md"
              >
                <CardContent>
                  <h3 className="font-bold mb-2 text-[#1F74DB]">
                    {status} ({recs.length})
                  </h3>
                  {renderTable(recs)}
                </CardContent>
              </Card>
            ))}
          </div>
        );
      default:
        return null;
    }
  };

  /* ===========================================================
     EARLY RETURN – INTRO SCREEN
     ===========================================================*/
  if (showIntro) {
    return <FlashIntroMain onStart={() => setShowIntro(false)} />;
  }

  /* ===========================================================
     MAIN JSX RENDER (Upload or Dashboard)
     ===========================================================*/
  return (
    <DndProvider backend={HTML5Backend}>
      <div className="p-6 w-full bg-gradient-to-b from-[#192234] to-[#092058] text-white min-h-screen">
        {/* ---------------- NO FILE YET – UPLOAD PAGE ---------------- */}
        {!uploaded ? (
          <div className="flex flex-col items-center justify-center min-h-screen bg-gradient-to-b from-[#192234] to-[#092058] text-white px-6 text-center">
            <h2 className="text-3xl md:text-4xl font-bold mb-10 animate-fade-in">
              Get Started in Three Simple Steps
            </h2>

            <div className="flex flex-col md:flex-row justify-center gap-6 mb-10 w-full max-w-7xl animate-slide-in-up">
              {[
                {
                  title: "Upload",
                  desc: "Drag and drop your annuity book file to begin the evaluation process."
                },
                {
                  title: "Analyze",
                  desc: "View contract-level insights, scoring, and priorities."
                },
                {
                  title: "Act",
                  desc: "Start meaningful client conversations based on what the data shows."
                }
              ].map((step, i) => (
                <div
                  key={i}
                  className="bg-transparent p-6 rounded-xl shadow-lg flex-1 min-w-[300px] max-w-sm transition-transform duration-300 hover:scale-105"
                >
                  <h3 className="text-xl font-bold mb-2">{step.title}</h3>
                  <p className="text-gray-300 text-sm">{step.desc}</p>
                </div>
              ))}
            </div>

            <div className="flex flex-col items-center gap-4 animate-fade-in delay-150">
              <label htmlFor="file-upload" className="bg-192234 hover:bg-blue-700 text-white font-semibold py-2 px-4 rounded cursor-pointer">
                Upload Book
              </label>
              <input
                id="file-upload"
                type="file"
                accept=".xlsx,.csv"
                onChange={handleFileUpload}
                className="hidden"
              />
            </div>

            {/* simple animations */}
            <style jsx>{`
              .animate-fade-in {
                animation: fadeIn 1s ease-in forwards;
                opacity: 0;
              }
              .animate-slide-in-up {
                animation: slideInUp 0.8s ease-out forwards;
                opacity: 0;
                transform: translateY(20px);
              }
              .delay-150 {
                animation-delay: 0.15s;
              }
              @keyframes fadeIn {
                to {
                  opacity: 1;
                }
              }
              @keyframes slideInUp {
                to {
                  opacity: 1;
                  transform: translateY(0);
                }
              }
            `}</style>
          </div>
        ) : (
          /* --------------------------- DASHBOARD --------------------------- */
          <>
            {/* top bar */}
<div className="flex justify-between mb-4 w-full">
  <div className="flex gap-2">
    <Button onClick={resetAll}>Upload New File</Button>
    
  </div>
  <TabButtons activeTab={activeTab} setActiveTab={setActiveTab} />
</div>
  


            {/* Quick metric cards & opportunity snapshot */}
            <div className="flex flex-wrap gap-4 mb-4">
              
{activeTab === "Wholesaler" && comparisonRequests.length > 0 && (
  <Card className="bg-yellow-100 text-black mb-4 p-4">
    <CardContent>
      <h4 className="font-bold mb-2 text-[#1F74DB]">📥 Advisor Adam Keighley Requests</h4>
      <ul className="list-disc list-inside text-sm">
        {comparisonRequests.map((name, i) => (
          <li key={i}>{name} ({requestScenarios[name] || 'Scenario not selected'}) comparison </li>
        ))}
      </ul>
    </CardContent>
  </Card>
)}

              {/* placeholder advisor info cards – could be dynamic */}
              {[
                { label: "Firm", value: "P.E.A.K. Advisory" },
                { label: "Advisor", value: "Adam Keighley" },
                { label: "Email", value: "Adam@PEAKRIA.COM" },
                { label: "Location", value: "Louisville, KY" }
              ].map((a, i) => (
                <DraggableCard key={a.label} id={a.label} index={i} moveCard={() => {}}>
                  <div
                    className="p-4 mb-4 text-gray-300"
                    style={{ width: "33%", minWidth: "280px", resize: "horizontal", overflow: "hidden" }}
                  >
                    <CardContent>
                      <h4 className="font-bold mb-1 text-[#1F74DB]">{a.label}</h4>
                      <p>{a.value}</p>
                    </CardContent>
                  </div>
                </DraggableCard>
              ))}
            </div>

            {/* MAIN SECTION AREA – two columns for charts, then tables */}
            <div className="w-full">
              <Card className="p-4 mb-4 bg-transparent text-white border border-white hover:shadow-xl transition-shadow duration-300 w-full backdrop-blur-md">
                <CardContent>
                  <div className="flex flex-col md:flex-row gap-4">
                    <div className="w-full md:w-[50%]">
                      {sections.includes("statusChart") && renderSection("statusChart")}
                    </div>
                    <div className="w-full md:w-[50%]">
                      {sections.includes("valueChart") && renderSection("valueChart")}
                    </div>
                  </div>
                </CardContent>
              </Card>
              <div className="flex flex-col md:flex-row gap-4">
                {sections
                  .filter((s) => s !== "statusChart" && s !== "valueChart")
                  .map((sec) => renderSection(sec))}
              </div>
            </div>

            {/* PROJECTION MODAL */}
            {showProjection && (
              <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center px-8 z-50">
                <div
                  className={`bg-white p-6 rounded-lg ${
                    expandedProjection ? "w-[90%] h-[90%]" : "w-full max-w-screen-lg"
                  }`}
                >
                  <div className="flex justify-between items-center mb-4">
                    <h3 className="text-2xl font-bold font-sans text-[#092058]">Projection</h3>
                    <Button
                      size="sm"
                      variant="outline"
                      onClick={() => setExpandedProjection(!expandedProjection)}
                    >
                      {expandedProjection ? "Collapse" : "Expand"}
                    </Button>
                  </div>

                  {/* controls */}
                  <div className="grid grid-cols-1 md:grid-cols-2 gap-4 mb-4 text-[#092058] font-semibold">
                    <div>
                      <label className="block mb-1">Projection Mode</label>
                      <select
                        className="bg-white border border-[#092058] px-2 py-1 rounded w-full"
                        value={projMode}
                        onChange={(e) => setProjMode(e.target.value)}
                      >
                        <option value="montecarlo">Monte Carlo</option>
                        <option value="fixed">Fixed Growth</option>
                      </select>
                    </div>
                    <div>
                      <label className="block mb-1">Compare to Nationwide</label>
                      <select
                        className="bg-white border border-[#092058] px-2 py-1 rounded w-full"
                        value={selectedScenario || ""}
                        onChange={(e) => setSelectedScenario(e.target.value)}
                      >
                        <option value="">-- None --</option>
                        {nationwideOptions.map((o) => (
                          <option key={o.id} value={o.id}>
                            {o.label}
                          </option>
                        ))}
                      </select>
                    </div>
                    <div>
                      <label className="block mb-1">Equity Allocation: {Math.round(allocAgg * 100)}%</label>
                      <input
                        type="range"
                        min="0"
                        max="100"
                        value={allocAgg * 100}
                        onChange={(e) => setAllocAgg(e.target.value / 100)}
                        className="w-full accent-[#092058]"
                      />
                    </div>
                    <div>
                      <label className="block mb-1">Income Start Year: {2025 + incomeStart}</label>
                      <input
                        type="range"
                        min="0"
                        max="30"
                        value={incomeStart}
                        onChange={(e) => setIncomeStart(Number(e.target.value))}
                        className="w-full accent-[#092058]"
                      />
                    </div>
                  </div>

                  {/* LINE CHART */}
     
<ResponsiveContainer width="100%" height={expandedProjection ? 500 : 300}>
  <LineChart data={projectionData} margin={{ top: 5, right: 30, left: 20, bottom: 5 }}>
    <XAxis dataKey="year" />
    <YAxis tickFormatter={(v) => `$${v.toLocaleString()}`} />
    <CartesianGrid strokeDasharray="3 3" />
    <RechartTooltip
  content={({ active, payload, label }) => {
    if (!active || !payload || payload.length === 0) return null;

    const current = payload.find(p => p.name === "Client Projection")?.payload;
    const compare = payload.find(p => p.name === "NARIA + CORE")?.payload;

    const format = (n) => `$${Number(n).toLocaleString()}`;
    const percent = (p) => `${(p * 100).toFixed(2)}%`;
    const sumIncomeYTD = (series, year) =>
      series.reduce((acc, pt) => acc + ((pt?.year <= year && pt?.income) || 0), 0);

    const clientAge = parseInt(data.find((r) => r["Client Name"] === selectedRows[0])?.["Client Age"]) || 65;
    const futureAge = clientAge + incomeStart;
    const nationwidePayout = getNariaPayoutRate(futureAge);

    return (
      <div className="p-4 bg-white border border-gray-400 rounded shadow-lg text-sm text-gray-800">
        <strong>Year {label}</strong>
        <div className="mt-2 mb-2 border-t border-gray-300" />

        {current && (
          <div className="mb-3">
            <strong className="block text-blue-600">Current Annuity</strong>
            Contract Value: {format(current.value)}<br />
            Benefit Base: {format(current.benefitBase)}<br />
            Income: {format(current.income)}<br />
            Return Rate: {percent(current.appliedReturn)}<br />
            <strong>Total Income:</strong> {format(sumIncomeYTD(projectionData, label))}<br />
<strong>Current Payout Rate:</strong> {percent(parseFloat(String(data.find((r) => r["Client Name"] === selectedRows[0])?.["Payout Rate"])?.replace(/%/g, "")) / 100)}
          </div>
        )}

        {compare && (
          <div>
            <strong className="block text-green-700">Nationwide Annuity</strong>
            Contract Value: {format(compare.value)}<br />
            Benefit Base: {format(compare.benefitBase)}<br />
            Income: {format(compare.income)}<br />
            Return Rate: {percent(compare.appliedReturn)}<br />
            <strong>Total Income:</strong> {format(sumIncomeYTD(comparisonData, label))}<br />
<strong>Nationwide Payout Rate:</strong> {percent(nationwidePayout)}
          </div>
        )}
      </div>
    );
  }}
/>

   <Line
      type="monotone"
      dataKey="value"
      stroke="#092058"
      dot={false}
      name="Client Projection"
    />

    {comparisonData.length > 0 && (
      <Line
        type="monotone"
        dataKey="value"
        stroke="#2E8B57"
        strokeDasharray="5 5"
        dot={false}
        data={comparisonData}
        name="NARIA + CORE"
      />
    )}
  </LineChart>
</ResponsiveContainer>
                   
                  <div className="flex justify-end mt-6">
                    <Button onClick={() => setShowProjection(false)}>Close</Button>
                  </div>
                </div>
              </div>
            )}
          </>
        )}
      </div>
    </DndProvider>
  );
}
Add AnnuityDemoTool component
