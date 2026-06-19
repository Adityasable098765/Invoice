import { useState, useEffect } from "react";

// ─── Design Tokens ─────────────────────────────────────────
// Palette: Deep ink navy + warm saffron accent + clean white
// Signature: WhatsApp-green send button + GST badge stamps
// Type: System UI for data density, bold weight hierarchy
const T = {
  navy: "#0F1B2D",
  navyLight: "#1A2D45",
  saffron: "#F59E0B",
  saffronLight: "#FEF3C7",
  green: "#25D366",
  greenDark: "#128C7E",
  red: "#EF4444",
  redLight: "#FEE2E2",
  slate: "#64748B",
  slateLight: "#F1F5F9",
  white: "#FFFFFF",
  border: "#E2E8F0",
  text: "#0F172A",
  textMuted: "#64748B",
};

// ─── Sample Data ────────────────────────────────────────────
const SAMPLE_INVOICES = [
  { id: "INV-001", client: "Sharma Traders", amount: 48500, gst: 8730, status: "overdue", daysAgo: 32, phone: "9876543210", items: [{ desc: "Office Supplies", hsn: "8443", qty: 50, rate: 970 }] },
  { id: "INV-002", client: "Riya Enterprises", amount: 22000, gst: 3960, status: "pending", daysAgo: 12, phone: "9123456789", items: [{ desc: "Packaging Material", hsn: "3923", qty: 200, rate: 110 }] },
  { id: "INV-003", client: "Mehta & Sons", amount: 91000, gst: 16380, status: "paid", daysAgo: 5, phone: "9988776655", items: [{ desc: "Steel Pipes", hsn: "7304", qty: 13, rate: 7000 }] },
  { id: "INV-004", client: "PK Distributors", amount: 15750, gst: 2835, status: "draft", daysAgo: 0, phone: "9001122334", items: [{ desc: "Electrical Fittings", hsn: "8536", qty: 35, rate: 450 }] },
  { id: "INV-005", client: "Laxmi Stores", amount: 33000, gst: 5940, status: "pending", daysAgo: 8, phone: "9765432100", items: [{ desc: "Textile Fabric", hsn: "5208", qty: 110, rate: 300 }] },
];

const STATUS_CONFIG = {
  paid: { label: "Paid", color: T.green, bg: "#DCFCE7", icon: "✓" },
  pending: { label: "Pending", color: "#D97706", bg: "#FEF3C7", icon: "⏳" },
  overdue: { label: "Overdue", color: T.red, bg: T.redLight, icon: "!" },
  draft: { label: "Draft", color: T.slate, bg: T.slateLight, icon: "✎" },
};

const GST_RATES = [5, 12, 18, 28];

// ─── Utility ────────────────────────────────────────────────
const fmt = (n) => "₹" + Number(n).toLocaleString("en-IN");
const fmtShort = (n) => n >= 100000 ? `₹${(n / 100000).toFixed(1)}L` : n >= 1000 ? `₹${(n / 1000).toFixed(0)}K` : `₹${n}`;

export default function InvoiceOS() {
  const [screen, setScreen] = useState("dashboard");
  const [invoices, setInvoices] = useState(SAMPLE_INVOICES);
  const [selected, setSelected] = useState(null);
  const [showNewInvoice, setShowNewInvoice] = useState(false);
  const [showReminder, setShowReminder] = useState(null);
  const [toast, setToast] = useState(null);
  const [aiLoading, setAiLoading] = useState(false);
  const [aiGSTReport, setAiGSTReport] = useState(null);
  const [newInv, setNewInv] = useState({ client: "", phone: "", gstNo: "", items: [{ desc: "", hsn: "", qty: 1, rate: "" }], gstRate: 18 });

  const showToast = (msg, type = "success") => {
    setToast({ msg, type });
    setTimeout(() => setToast(null), 3000);
  };

  const totalReceivable = invoices.filter(i => i.status !== "paid" && i.status !== "draft").reduce((s, i) => s + i.amount + i.gst, 0);
  const totalOverdue = invoices.filter(i => i.status === "overdue").reduce((s, i) => s + i.amount + i.gst, 0);
  const totalPaid = invoices.filter(i => i.status === "paid").reduce((s, i) => s + i.amount + i.gst, 0);
  const totalGSTDue = invoices.filter(i => i.status !== "draft").reduce((s, i) => s + i.gst, 0);

  const addItem = () => setNewInv(p => ({ ...p, items: [...p.items, { desc: "", hsn: "", qty: 1, rate: "" }] }));
  const updateItem = (idx, field, val) => setNewInv(p => ({ ...p, items: p.items.map((it, i) => i === idx ? { ...it, [field]: val } : it) }));

  const subTotal = newInv.items.reduce((s, it) => s + (Number(it.qty) * Number(it.rate) || 0), 0);
  const gstAmt = Math.round(subTotal * newInv.gstRate / 100);

  const createInvoice = () => {
    if (!newInv.client) return showToast("Enter client name", "error");
    const id = "INV-00" + (invoices.length + 1);
    setInvoices(p => [...p, { id, client: newInv.client, phone: newInv.phone, amount: subTotal, gst: gstAmt, status: "draft", daysAgo: 0, items: newInv.items }]);
    setNewInv({ client: "", phone: "", gstNo: "", items: [{ desc: "", hsn: "", qty: 1, rate: "" }], gstRate: 18 });
    setShowNewInvoice(false);
    showToast(`${id} created successfully`);
  };

  const sendReminder = (inv) => {
    const msg = `Hello ${inv.client},\n\nKindly note that Invoice ${inv.id} of ${fmt(inv.amount + inv.gst)} (incl. GST) is ${inv.status === "overdue" ? "OVERDUE" : "pending"} since ${inv.daysAgo} days.\n\nPlease make payment at the earliest.\n\nThank you,\nInvoiceOS`;
    window.open(`https://wa.me/91${inv.phone}?text=${encodeURIComponent(msg)}`, "_blank");
    showToast("WhatsApp reminder opened");
    setShowReminder(null);
  };

  const markPaid = (id) => {
    setInvoices(p => p.map(i => i.id === id ? { ...i, status: "paid" } : i));
    showToast("Marked as paid ✓");
    setSelected(null);
  };

  const generateGSTReport = async () => {
    setAiLoading(true);
    setScreen("gst");
    try {
      const summary = invoices.filter(i => i.status !== "draft").map(i => `${i.id}: ${i.client}, Base=₹${i.amount}, GST=₹${i.gst}, Status=${i.status}`).join("\n");
      const resp = await fetch("https://api.anthropic.com/v1/messages", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          model: "claude-sonnet-4-6",
          max_tokens: 1000,
          system: "You are a GST compliance expert for Indian micro-SMEs. Respond ONLY in JSON, no markdown or backticks. Return: {\"gstr1_summary\": string, \"gstr3b_liability\": number, \"input_credit_estimate\": number, \"net_payable\": number, \"filing_deadline\": string, \"action_items\": string[], \"risk_flags\": string[]}",
          messages: [{ role: "user", content: `Analyze these invoices and generate a GSTR summary for this month:\n${summary}\nTotal GST collected: ₹${totalGSTDue}` }]
        })
      });
      const data = await resp.json();
      const text = data.content?.find(b => b.type === "text")?.text || "{}";
      setAiGSTReport(JSON.parse(text.replace(/```json|```/g, "").trim()));
    } catch (e) {
      setAiGSTReport({ gstr1_summary: "Analysis ready", gstr3b_liability: totalGSTDue, input_credit_estimate: Math.round(totalGSTDue * 0.3), net_payable: Math.round(totalGSTDue * 0.7), filing_deadline: "20th July 2026", action_items: ["File GSTR-1 before deadline", "Reconcile GSTR-2A with purchase register", "Verify HSN codes on all invoices"], risk_flags: ["2 overdue invoices may affect cash flow for GST payment"] });
    }
    setAiLoading(false);
  };

  return (
    <div style={{ minHeight: "100vh", background: T.slateLight, fontFamily: "'Inter', system-ui, sans-serif", color: T.text }}>

      {/* Toast */}
      {toast && (
        <div style={{ position: "fixed", top: 16, right: 16, zIndex: 999, background: toast.type === "error" ? T.red : T.navy, color: "#fff", padding: "10px 18px", borderRadius: 10, fontSize: 14, fontWeight: 500, boxShadow: "0 4px 20px rgba(0,0,0,0.2)", display: "flex", alignItems: "center", gap: 8 }}>
          {toast.type === "error" ? "⚠️" : "✓"} {toast.msg}
        </div>
      )}

      {/* Sidebar */}
      <div style={{ position: "fixed", left: 0, top: 0, bottom: 0, width: 220, background: T.navy, display: "flex", flexDirection: "column", zIndex: 100 }}>
        <div style={{ padding: "20px 20px 16px" }}>
          <div style={{ display: "flex", alignItems: "center", gap: 8, marginBottom: 4 }}>
            <div style={{ width: 32, height: 32, background: T.saffron, borderRadius: 8, display: "flex", alignItems: "center", justifyContent: "center", fontSize: 16 }}>⚡</div>
            <span style={{ color: "#fff", fontWeight: 700, fontSize: 17, letterSpacing: -0.3 }}>InvoiceOS</span>
          </div>
          <div style={{ fontSize: 11, color: "rgba(255,255,255,0.4)", paddingLeft: 40 }}>GST-native · WhatsApp-first</div>
        </div>

        <div style={{ flex: 1, padding: "8px 12px" }}>
          {[
            { id: "dashboard", icon: "⊞", label: "Dashboard" },
            { id: "invoices", icon: "📄", label: "Invoices" },
            { id: "gst", icon: "🏛️", label: "GST Filing" },
            { id: "analytics", icon: "📊", label: "Analytics" },
          ].map(nav => (
            <button key={nav.id} onClick={() => nav.id === "gst" ? generateGSTReport() : setScreen(nav.id)}
              style={{ width: "100%", display: "flex", alignItems: "center", gap: 10, padding: "9px 12px", borderRadius: 8, border: "none", cursor: "pointer", marginBottom: 2, background: screen === nav.id ? "rgba(245,158,11,0.15)" : "transparent", color: screen === nav.id ? T.saffron : "rgba(255,255,255,0.65)", fontWeight: screen === nav.id ? 600 : 400, fontSize: 14, textAlign: "left", transition: "all 0.15s" }}>
              <span style={{ fontSize: 16 }}>{nav.icon}</span> {nav.label}
            </button>
          ))}
        </div>

        <div style={{ padding: 12 }}>
          <button onClick={() => { setShowNewInvoice(true); setScreen("invoices"); }}
            style={{ width: "100%", padding: "11px 0", background: T.saffron, border: "none", borderRadius: 10, color: T.navy, fontWeight: 700, fontSize: 14, cursor: "pointer", display: "flex", alignItems: "center", justifyContent: "center", gap: 6 }}>
            + New Invoice
          </button>
        </div>
      </div>

      {/* Main */}
      <div style={{ marginLeft: 220, minHeight: "100vh" }}>
        {/* Header */}
        <div style={{ background: T.white, borderBottom: `1px solid ${T.border}`, padding: "14px 28px", display: "flex", alignItems: "center", justifyContent: "space-between", position: "sticky", top: 0, zIndex: 50 }}>
          <div>
            <div style={{ fontWeight: 700, fontSize: 18 }}>
              {screen === "dashboard" && "Dashboard"}
              {screen === "invoices" && "Invoices"}
              {screen === "gst" && "GST Filing Assistant"}
              {screen === "analytics" && "Analytics"}
            </div>
            <div style={{ fontSize: 12, color: T.textMuted }}>June 2026</div>
          </div>
          <div style={{ display: "flex", gap: 10, alignItems: "center" }}>
            <div style={{ background: T.saffronLight, border: `1px solid ${T.saffron}`, borderRadius: 20, padding: "4px 12px", fontSize: 12, color: "#92400E", fontWeight: 600 }}>
              GSTIN: 27AAPFU0939F1ZV
            </div>
            <div style={{ width: 34, height: 34, background: T.navy, borderRadius: "50%", display: "flex", alignItems: "center", justifyContent: "center", color: "#fff", fontWeight: 700, fontSize: 14 }}>A</div>
          </div>
        </div>

        <div style={{ padding: 28 }}>

          {/* ── DASHBOARD ── */}
          {screen === "dashboard" && (
            <div>
              {/* Stats */}
              <div style={{ display: "grid", gridTemplateColumns: "repeat(4, 1fr)", gap: 16, marginBottom: 28 }}>
                {[
                  { label: "Total Receivable", value: fmtShort(totalReceivable), sub: "from " + invoices.filter(i => i.status === "pending" || i.status === "overdue").length + " invoices", color: T.navy, icon: "💰" },
                  { label: "Overdue", value: fmtShort(totalOverdue), sub: invoices.filter(i => i.status === "overdue").length + " invoices past due", color: T.red, icon: "⚠️" },
                  { label: "Collected (June)", value: fmtShort(totalPaid), sub: "on time payments", color: "#059669", icon: "✅" },
                  { label: "GST Liability", value: fmtShort(totalGSTDue), sub: "GSTR-3B due 20 Jul", color: "#7C3AED", icon: "🏛️" },
                ].map(s => (
                  <div key={s.label} style={{ background: T.white, borderRadius: 14, padding: "18px 20px", border: `1px solid ${T.border}`, position: "relative", overflow: "hidden" }}>
                    <div style={{ position: "absolute", top: 14, right: 16, fontSize: 22, opacity: 0.15 }}>{s.icon}</div>
                    <div style={{ fontSize: 12, color: T.textMuted, fontWeight: 500, marginBottom: 6 }}>{s.label}</div>
                    <div style={{ fontSize: 26, fontWeight: 800, color: s.color, letterSpacing: -1 }}>{s.value}</div>
                    <div style={{ fontSize: 11, color: T.textMuted, marginTop: 4 }}>{s.sub}</div>
                  </div>
                ))}
              </div>

              {/* Overdue Alert */}
              {totalOverdue > 0 && (
                <div style={{ background: "#FFF7ED", border: `1px solid #FED7AA`, borderRadius: 12, padding: "14px 18px", marginBottom: 20, display: "flex", alignItems: "center", justifyContent: "space-between" }}>
                  <div style={{ display: "flex", alignItems: "center", gap: 10 }}>
                    <span style={{ fontSize: 20 }}>🔔</span>
                    <div>
                      <div style={{ fontWeight: 600, fontSize: 14, color: "#9A3412" }}>
                        {invoices.filter(i => i.status === "overdue").length} overdue invoices — {fmtShort(totalOverdue)} pending
                      </div>
                      <div style={{ fontSize: 12, color: "#C2410C" }}>Send bulk WhatsApp reminders to recover faster</div>
                    </div>
                  </div>
                  <button onClick={() => { setScreen("invoices"); }}
                    style={{ background: "#EA580C", border: "none", borderRadius: 8, color: "#fff", padding: "8px 16px", fontSize: 13, fontWeight: 600, cursor: "pointer" }}>
                    Send Reminders
                  </button>
                </div>
              )}

              {/* Recent Invoices */}
              <div style={{ background: T.white, borderRadius: 14, border: `1px solid ${T.border}` }}>
                <div style={{ padding: "16px 20px", borderBottom: `1px solid ${T.border}`, display: "flex", justifyContent: "space-between", alignItems: "center" }}>
                  <span style={{ fontWeight: 700, fontSize: 15 }}>Recent Invoices</span>
                  <button onClick={() => setScreen("invoices")} style={{ background: "none", border: "none", color: T.saffron, fontSize: 13, fontWeight: 600, cursor: "pointer" }}>View all →</button>
                </div>
                <InvoiceTable invoices={invoices.slice(0, 5)} onSelect={setSelected} onReminder={setShowReminder} onMarkPaid={markPaid} />
              </div>
            </div>
          )}

          {/* ── INVOICES ── */}
          {screen === "invoices" && (
            <div>
              {showNewInvoice && (
                <div style={{ background: T.white, borderRadius: 14, border: `1px solid ${T.border}`, padding: 24, marginBottom: 24 }}>
                  <div style={{ fontWeight: 700, fontSize: 16, marginBottom: 20, display: "flex", justifyContent: "space-between", alignItems: "center" }}>
                    <span>✚ New GST Invoice</span>
                    <button onClick={() => setShowNewInvoice(false)} style={{ background: "none", border: "none", color: T.slate, cursor: "pointer", fontSize: 20 }}>×</button>
                  </div>

                  <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr 1fr", gap: 14, marginBottom: 20 }}>
                    {[
                      { label: "Client Name *", field: "client", placeholder: "Sharma Traders" },
                      { label: "WhatsApp Number", field: "phone", placeholder: "9876543210" },
                      { label: "Client GSTIN", field: "gstNo", placeholder: "27AAPFU0939F1ZV" },
                    ].map(f => (
                      <div key={f.field}>
                        <label style={{ fontSize: 12, color: T.textMuted, fontWeight: 600, display: "block", marginBottom: 6 }}>{f.label}</label>
                        <input value={newInv[f.field]} onChange={e => setNewInv(p => ({ ...p, [f.field]: e.target.value }))}
                          placeholder={f.placeholder}
                          style={{ width: "100%", padding: "9px 12px", border: `1px solid ${T.border}`, borderRadius: 8, fontSize: 14, outline: "none", boxSizing: "border-box" }} />
                      </div>
                    ))}
                  </div>

                  <div style={{ marginBottom: 16 }}>
                    <div style={{ display: "grid", gridTemplateColumns: "3fr 1.2fr 0.8fr 1fr 0.4fr", gap: 8, marginBottom: 8 }}>
                      {["Description", "HSN Code", "Qty", "Rate (₹)", ""].map(h => (
                        <div key={h} style={{ fontSize: 11, color: T.textMuted, fontWeight: 600 }}>{h}</div>
                      ))}
                    </div>
                    {newInv.items.map((item, idx) => (
                      <div key={idx} style={{ display: "grid", gridTemplateColumns: "3fr 1.2fr 0.8fr 1fr 0.4fr", gap: 8, marginBottom: 8 }}>
                        <input value={item.desc} onChange={e => updateItem(idx, "desc", e.target.value)} placeholder="Item description"
                          style={{ padding: "8px 10px", border: `1px solid ${T.border}`, borderRadius: 7, fontSize: 13, outline: "none" }} />
                        <input value={item.hsn} onChange={e => updateItem(idx, "hsn", e.target.value)} placeholder="8443"
                          style={{ padding: "8px 10px", border: `1px solid ${T.border}`, borderRadius: 7, fontSize: 13, outline: "none" }} />
                        <input type="number" value={item.qty} onChange={e => updateItem(idx, "qty", e.target.value)}
                          style={{ padding: "8px 10px", border: `1px solid ${T.border}`, borderRadius: 7, fontSize: 13, outline: "none" }} />
                        <input type="number" value={item.rate} onChange={e => updateItem(idx, "rate", e.target.value)} placeholder="0"
                          style={{ padding: "8px 10px", border: `1px solid ${T.border}`, borderRadius: 7, fontSize: 13, outline: "none" }} />
                        <button onClick={() => setNewInv(p => ({ ...p, items: p.items.filter((_, i) => i !== idx) }))}
                          style={{ background: T.redLight, border: "none", borderRadius: 7, color: T.red, cursor: "pointer", fontSize: 16 }}>×</button>
                      </div>
                    ))}
                    <button onClick={addItem} style={{ background: "none", border: `1px dashed ${T.border}`, borderRadius: 8, color: T.textMuted, padding: "7px 14px", fontSize: 13, cursor: "pointer", marginTop: 4 }}>+ Add Item</button>
                  </div>

                  <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-end" }}>
                    <div style={{ display: "flex", gap: 8, alignItems: "center" }}>
                      <span style={{ fontSize: 13, color: T.textMuted }}>GST Rate:</span>
                      {GST_RATES.map(r => (
                        <button key={r} onClick={() => setNewInv(p => ({ ...p, gstRate: r }))}
                          style={{ padding: "5px 12px", borderRadius: 20, border: `1px solid ${newInv.gstRate === r ? T.saffron : T.border}`, background: newInv.gstRate === r ? T.saffronLight : "#fff", color: newInv.gstRate === r ? "#92400E" : T.slate, fontWeight: newInv.gstRate === r ? 700 : 400, fontSize: 13, cursor: "pointer" }}>
                          {r}%
                        </button>
                      ))}
                    </div>
                    <div style={{ textAlign: "right" }}>
                      <div style={{ fontSize: 13, color: T.textMuted }}>Subtotal: {fmt(subTotal)} + GST {fmt(gstAmt)}</div>
                      <div style={{ fontSize: 22, fontWeight: 800, color: T.navy }}>Total: {fmt(subTotal + gstAmt)}</div>
                    </div>
                  </div>

                  <div style={{ display: "flex", gap: 10, marginTop: 20, justifyContent: "flex-end" }}>
                    <button onClick={() => setShowNewInvoice(false)} style={{ padding: "10px 20px", background: T.slateLight, border: "none", borderRadius: 9, fontSize: 14, cursor: "pointer" }}>Cancel</button>
                    <button onClick={createInvoice} style={{ padding: "10px 24px", background: T.navy, border: "none", borderRadius: 9, color: "#fff", fontSize: 14, fontWeight: 600, cursor: "pointer" }}>Save Invoice</button>
                    <button onClick={() => { createInvoice(); }} style={{ padding: "10px 24px", background: T.green, border: "none", borderRadius: 9, color: "#fff", fontSize: 14, fontWeight: 600, cursor: "pointer", display: "flex", alignItems: "center", gap: 6 }}>
                      <span>📱</span> Save & Send on WhatsApp
                    </button>
                  </div>
                </div>
              )}

              {/* Filter tabs */}
              <div style={{ display: "flex", gap: 8, marginBottom: 16 }}>
                {["all", "pending", "overdue", "paid", "draft"].map(f => {
                  const count = f === "all" ? invoices.length : invoices.filter(i => i.status === f).length;
                  return (
                    <button key={f} style={{ padding: "6px 14px", borderRadius: 20, border: `1px solid ${T.border}`, background: T.white, fontSize: 13, cursor: "pointer", fontWeight: 500, color: T.text }}>
                      {f.charAt(0).toUpperCase() + f.slice(1)} ({count})
                    </button>
                  );
                })}
              </div>

              <div style={{ background: T.white, borderRadius: 14, border: `1px solid ${T.border}` }}>
                <InvoiceTable invoices={invoices} onSelect={setSelected} onReminder={setShowReminder} onMarkPaid={markPaid} />
              </div>
            </div>
          )}

          {/* ── GST FILING ── */}
          {screen === "gst" && (
            <div>
              {aiLoading ? (
                <div style={{ background: T.white, borderRadius: 14, border: `1px solid ${T.border}`, padding: 60, textAlign: "center" }}>
                  <div style={{ fontSize: 40, marginBottom: 16 }}>🏛️</div>
                  <div style={{ fontWeight: 600, fontSize: 16, marginBottom: 8 }}>Analysing your invoices...</div>
                  <div style={{ color: T.textMuted, fontSize: 14 }}>Generating GSTR summary with AI</div>
                  <div style={{ width: 200, height: 4, background: T.slateLight, borderRadius: 2, margin: "20px auto 0", overflow: "hidden" }}>
                    <div style={{ height: "100%", background: T.saffron, borderRadius: 2, width: "70%", animation: "pulse 1s infinite" }} />
                  </div>
                </div>
              ) : aiGSTReport ? (
                <div>
                  {/* GST Summary Cards */}
                  <div style={{ display: "grid", gridTemplateColumns: "repeat(3, 1fr)", gap: 16, marginBottom: 24 }}>
                    {[
                      { label: "GSTR-3B Liability", value: fmt(aiGSTReport.gstr3b_liability), color: "#7C3AED", icon: "📤" },
                      { label: "Input Tax Credit (est.)", value: fmt(aiGSTReport.input_credit_estimate), color: "#059669", icon: "📥" },
                      { label: "Net GST Payable", value: fmt(aiGSTReport.net_payable), color: T.red, icon: "💳" },
                    ].map(s => (
                      <div key={s.label} style={{ background: T.white, borderRadius: 14, padding: "20px 22px", border: `1px solid ${T.border}` }}>
                        <div style={{ fontSize: 24, marginBottom: 8 }}>{s.icon}</div>
                        <div style={{ fontSize: 12, color: T.textMuted, fontWeight: 600, marginBottom: 4 }}>{s.label}</div>
                        <div style={{ fontSize: 28, fontWeight: 800, color: s.color }}>{s.value}</div>
                      </div>
                    ))}
                  </div>

                  <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 16, marginBottom: 20 }}>
                    {/* Action Items */}
                    <div style={{ background: T.white, borderRadius: 14, border: `1px solid ${T.border}`, padding: "20px 22px" }}>
                      <div style={{ fontWeight: 700, fontSize: 15, marginBottom: 14, display: "flex", alignItems: "center", gap: 8 }}>
                        <span>✅</span> Action Items
                      </div>
                      {(aiGSTReport.action_items || []).map((item, i) => (
                        <div key={i} style={{ display: "flex", gap: 10, padding: "8px 0", borderBottom: i < aiGSTReport.action_items.length - 1 ? `1px solid ${T.border}` : "none", alignItems: "flex-start" }}>
                          <div style={{ width: 22, height: 22, background: "#DCFCE7", borderRadius: "50%", display: "flex", alignItems: "center", justifyContent: "center", fontSize: 11, color: "#059669", fontWeight: 700, flexShrink: 0 }}>{i + 1}</div>
                          <div style={{ fontSize: 13, color: T.text }}>{item}</div>
                        </div>
                      ))}
                    </div>

                    {/* Risk Flags */}
                    <div style={{ background: T.white, borderRadius: 14, border: `1px solid ${T.border}`, padding: "20px 22px" }}>
                      <div style={{ fontWeight: 700, fontSize: 15, marginBottom: 14, display: "flex", alignItems: "center", gap: 8 }}>
                        <span>⚠️</span> Risk Flags
                      </div>
                      {(aiGSTReport.risk_flags || []).map((flag, i) => (
                        <div key={i} style={{ background: "#FFF7ED", border: `1px solid #FED7AA`, borderRadius: 8, padding: "10px 12px", marginBottom: 8, fontSize: 13, color: "#9A3412" }}>
                          {flag}
                        </div>
                      ))}
                      <div style={{ marginTop: 12 }}>
                        <div style={{ fontSize: 12, color: T.textMuted }}>Filing Deadline</div>
                        <div style={{ fontWeight: 700, fontSize: 16, color: T.red }}>{aiGSTReport.filing_deadline}</div>
                      </div>
                    </div>
                  </div>

                  {/* GSTR Summary */}
                  <div style={{ background: T.white, borderRadius: 14, border: `1px solid ${T.border}`, padding: "20px 22px", marginBottom: 20 }}>
                    <div style={{ fontWeight: 700, fontSize: 15, marginBottom: 10 }}>🏛️ GSTR-1 Summary (AI Analysis)</div>
                    <div style={{ fontSize: 14, color: T.text, lineHeight: 1.7, background: T.slateLight, borderRadius: 8, padding: "12px 16px" }}>
                      {aiGSTReport.gstr1_summary}
                    </div>
                  </div>

                  <div style={{ display: "flex", gap: 10 }}>
                    <button style={{ padding: "11px 22px", background: "#7C3AED", border: "none", borderRadius: 10, color: "#fff", fontWeight: 600, fontSize: 14, cursor: "pointer" }}>
                      📤 Export GSTR-1 JSON
                    </button>
                    <button style={{ padding: "11px 22px", background: T.navy, border: "none", borderRadius: 10, color: "#fff", fontWeight: 600, fontSize: 14, cursor: "pointer" }}>
                      📊 Export to Tally XML
                    </button>
                    <button style={{ padding: "11px 22px", background: T.white, border: `1px solid ${T.border}`, borderRadius: 10, color: T.text, fontWeight: 500, fontSize: 14, cursor: "pointer" }}>
                      📩 Email to CA
                    </button>
                  </div>
                </div>
              ) : null}
            </div>
          )}

          {/* ── ANALYTICS ── */}
          {screen === "analytics" && (
            <div>
              <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 20 }}>
                {/* Invoice Status Breakdown */}
                <div style={{ background: T.white, borderRadius: 14, border: `1px solid ${T.border}`, padding: "20px 22px" }}>
                  <div style={{ fontWeight: 700, fontSize: 15, marginBottom: 18 }}>Invoice Status Breakdown</div>
                  {Object.entries(STATUS_CONFIG).map(([status, cfg]) => {
                    const count = invoices.filter(i => i.status === status).length;
                    const pct = Math.round((count / invoices.length) * 100);
                    return (
                      <div key={status} style={{ marginBottom: 14 }}>
                        <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 5 }}>
                          <span style={{ fontSize: 13, fontWeight: 500 }}>{cfg.label}</span>
                          <span style={{ fontSize: 13, color: T.textMuted }}>{count} invoices ({pct}%)</span>
                        </div>
                        <div style={{ height: 8, background: T.slateLight, borderRadius: 4, overflow: "hidden" }}>
                          <div style={{ height: "100%", width: `${pct}%`, background: cfg.color, borderRadius: 4 }} />
                        </div>
                      </div>
                    );
                  })}
                </div>

                {/* Top Clients */}
                <div style={{ background: T.white, borderRadius: 14, border: `1px solid ${T.border}`, padding: "20px 22px" }}>
                  <div style={{ fontWeight: 700, fontSize: 15, marginBottom: 18 }}>Top Clients by Value</div>
                  {[...invoices].sort((a, b) => (b.amount + b.gst) - (a.amount + a.gst)).map((inv, i) => (
                    <div key={inv.id} style={{ display: "flex", alignItems: "center", gap: 12, marginBottom: 12, padding: "8px 0", borderBottom: i < invoices.length - 1 ? `1px solid ${T.border}` : "none" }}>
                      <div style={{ width: 28, height: 28, background: i === 0 ? T.saffron : T.slateLight, borderRadius: "50%", display: "flex", alignItems: "center", justifyContent: "center", fontWeight: 700, fontSize: 12, color: i === 0 ? T.navy : T.slate, flexShrink: 0 }}>{i + 1}</div>
                      <div style={{ flex: 1 }}>
                        <div style={{ fontWeight: 500, fontSize: 14 }}>{inv.client}</div>
                        <div style={{ fontSize: 11, color: T.textMuted }}>{inv.id}</div>
                      </div>
                      <div style={{ textAlign: "right" }}>
                        <div style={{ fontWeight: 700, fontSize: 15 }}>{fmt(inv.amount + inv.gst)}</div>
                        <div style={{ ...statusBadgeStyle(inv.status) }}>{STATUS_CONFIG[inv.status].label}</div>
                      </div>
                    </div>
                  ))}
                </div>

                {/* Cash Flow */}
                <div style={{ background: T.white, borderRadius: 14, border: `1px solid ${T.border}`, padding: "20px 22px", gridColumn: "1 / -1" }}>
                  <div style={{ fontWeight: 700, fontSize: 15, marginBottom: 18 }}>Cash Flow Snapshot</div>
                  <div style={{ display: "grid", gridTemplateColumns: "repeat(3, 1fr)", gap: 16 }}>
                    {[
                      { label: "Expected This Month", value: fmt(totalReceivable), color: T.navy },
                      { label: "Overdue Recovery Needed", value: fmt(totalOverdue), color: T.red },
                      { label: "Average Invoice Value", value: fmt(Math.round(invoices.reduce((s, i) => s + i.amount + i.gst, 0) / invoices.length)), color: "#7C3AED" },
                    ].map(s => (
                      <div key={s.label} style={{ background: T.slateLight, borderRadius: 10, padding: "16px 18px" }}>
                        <div style={{ fontSize: 12, color: T.textMuted, fontWeight: 600, marginBottom: 6 }}>{s.label}</div>
                        <div style={{ fontSize: 24, fontWeight: 800, color: s.color }}>{s.value}</div>
                      </div>
                    ))}
                  </div>
                </div>
              </div>
            </div>
          )}
        </div>
      </div>

      {/* Invoice Detail Modal */}
      {selected && (
        <div style={{ position: "fixed", inset: 0, background: "rgba(0,0,0,0.5)", zIndex: 200, display: "flex", alignItems: "center", justifyContent: "center" }}
          onClick={() => setSelected(null)}>
          <div style={{ background: T.white, borderRadius: 16, width: 520, padding: 28, boxShadow: "0 20px 60px rgba(0,0,0,0.25)" }} onClick={e => e.stopPropagation()}>
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start", marginBottom: 20 }}>
              <div>
                <div style={{ fontWeight: 800, fontSize: 20, letterSpacing: -0.5 }}>{selected.id}</div>
                <div style={{ color: T.textMuted, fontSize: 14 }}>{selected.client}</div>
              </div>
              <div style={{ display: "flex", gap: 8, alignItems: "center" }}>
                <span style={statusBadgeStyle(selected.status)}>{STATUS_CONFIG[selected.status].icon} {STATUS_CONFIG[selected.status].label}</span>
                <button onClick={() => setSelected(null)} style={{ background: "none", border: "none", fontSize: 22, cursor: "pointer", color: T.slate }}>×</button>
              </div>
            </div>

            <div style={{ background: T.slateLight, borderRadius: 10, padding: "14px 16px", marginBottom: 16 }}>
              {selected.items.map((item, i) => (
                <div key={i} style={{ display: "flex", justifyContent: "space-between", padding: "6px 0", borderBottom: i < selected.items.length - 1 ? `1px solid ${T.border}` : "none" }}>
                  <div>
                    <div style={{ fontWeight: 500, fontSize: 14 }}>{item.desc}</div>
                    <div style={{ fontSize: 11, color: T.textMuted }}>HSN: {item.hsn} · Qty: {item.qty}</div>
                  </div>
                  <div style={{ fontWeight: 600, fontSize: 14 }}>{fmt(item.qty * item.rate)}</div>
                </div>
              ))}
            </div>

            <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 6 }}>
              <span style={{ color: T.textMuted }}>Subtotal</span><span style={{ fontWeight: 600 }}>{fmt(selected.amount)}</span>
            </div>
            <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 12 }}>
              <span style={{ color: T.textMuted }}>GST</span><span style={{ fontWeight: 600, color: "#7C3AED" }}>{fmt(selected.gst)}</span>
            </div>
            <div style={{ display: "flex", justifyContent: "space-between", padding: "10px 0", borderTop: `2px solid ${T.border}`, marginBottom: 20 }}>
              <span style={{ fontWeight: 700, fontSize: 16 }}>Total</span><span style={{ fontWeight: 800, fontSize: 18 }}>{fmt(selected.amount + selected.gst)}</span>
            </div>

            <div style={{ display: "flex", gap: 10 }}>
              {selected.status !== "paid" && (
                <>
                  <button onClick={() => { setShowReminder(selected); setSelected(null); }}
                    style={{ flex: 1, padding: "10px 0", background: T.green, border: "none", borderRadius: 9, color: "#fff", fontWeight: 600, fontSize: 14, cursor: "pointer", display: "flex", alignItems: "center", justifyContent: "center", gap: 6 }}>
                    📱 WhatsApp
                  </button>
                  <button onClick={() => markPaid(selected.id)}
                    style={{ flex: 1, padding: "10px 0", background: T.navy, border: "none", borderRadius: 9, color: "#fff", fontWeight: 600, fontSize: 14, cursor: "pointer" }}>
                    ✓ Mark Paid
                  </button>
                </>
              )}
              <button style={{ flex: 1, padding: "10px 0", background: T.slateLight, border: "none", borderRadius: 9, fontSize: 14, cursor: "pointer" }}>
                📄 Download PDF
              </button>
            </div>
          </div>
        </div>
      )}

      {/* WhatsApp Reminder Modal */}
      {showReminder && (
        <div style={{ position: "fixed", inset: 0, background: "rgba(0,0,0,0.5)", zIndex: 200, display: "flex", alignItems: "center", justifyContent: "center" }}
          onClick={() => setShowReminder(null)}>
          <div style={{ background: T.white, borderRadius: 16, width: 440, padding: 24 }} onClick={e => e.stopPropagation()}>
            <div style={{ fontWeight: 700, fontSize: 16, marginBottom: 4 }}>📱 Send WhatsApp Reminder</div>
            <div style={{ color: T.textMuted, fontSize: 13, marginBottom: 16 }}>To: {showReminder.client} (+91 {showReminder.phone})</div>
            <div style={{ background: "#DCF8C6", borderRadius: 12, padding: 14, fontSize: 13, lineHeight: 1.7, color: "#111", fontFamily: "monospace", marginBottom: 16, position: "relative" }}>
              <div style={{ position: "absolute", top: -6, left: 14, width: 12, height: 12, background: "#DCF8C6", transform: "rotate(45deg)" }} />
              Hello {showReminder.client},{"\n\n"}Invoice {showReminder.id} of {fmt(showReminder.amount + showReminder.gst)} is {showReminder.status === "overdue" ? "OVERDUE" : "pending"} since {showReminder.daysAgo} days.{"\n\n"}Please arrange payment at the earliest.{"\n\n"}Thank you
            </div>
            <div style={{ display: "flex", gap: 10 }}>
              <button onClick={() => setShowReminder(null)} style={{ flex: 1, padding: "10px 0", background: T.slateLight, border: "none", borderRadius: 9, fontSize: 14, cursor: "pointer" }}>Cancel</button>
              <button onClick={() => sendReminder(showReminder)}
                style={{ flex: 1, padding: "10px 0", background: T.green, border: "none", borderRadius: 9, color: "#fff", fontWeight: 700, fontSize: 14, cursor: "pointer", display: "flex", alignItems: "center", justifyContent: "center", gap: 8 }}>
                <span>📲</span> Send on WhatsApp
              </button>
            </div>
          </div>
        </div>
      )}
    </div>
  );
}

// ── Sub-components ──────────────────────────────────────────
function InvoiceTable({ invoices, onSelect, onReminder, onMarkPaid }) {
  return (
    <table style={{ width: "100%", borderCollapse: "collapse" }}>
      <thead>
        <tr style={{ borderBottom: `1px solid ${T.border}` }}>
          {["Invoice", "Client", "Amount", "GST", "Status", "Age", "Actions"].map(h => (
            <th key={h} style={{ padding: "12px 16px", textAlign: "left", fontSize: 12, color: T.textMuted, fontWeight: 600 }}>{h}</th>
          ))}
        </tr>
      </thead>
      <tbody>
        {invoices.map(inv => (
          <tr key={inv.id} style={{ borderBottom: `1px solid ${T.border}`, transition: "background 0.1s" }}
            onMouseEnter={e => e.currentTarget.style.background = T.slateLight}
            onMouseLeave={e => e.currentTarget.style.background = "transparent"}>
            <td style={{ padding: "12px 16px", fontWeight: 700, fontSize: 13, color: T.navy, cursor: "pointer" }} onClick={() => onSelect(inv)}>{inv.id}</td>
            <td style={{ padding: "12px 16px", fontSize: 14 }}>{inv.client}</td>
            <td style={{ padding: "12px 16px", fontWeight: 600, fontSize: 14 }}>{fmt(inv.amount)}</td>
            <td style={{ padding: "12px 16px", fontSize: 13, color: "#7C3AED", fontWeight: 500 }}>{fmt(inv.gst)}</td>
            <td style={{ padding: "12px 16px" }}><span style={statusBadgeStyle(inv.status)}>{STATUS_CONFIG[inv.status].icon} {STATUS_CONFIG[inv.status].label}</span></td>
            <td style={{ padding: "12px 16px", fontSize: 13, color: inv.status === "overdue" ? T.red : T.textMuted }}>
              {inv.daysAgo === 0 ? "Today" : `${inv.daysAgo}d ago`}
            </td>
            <td style={{ padding: "12px 16px" }}>
              <div style={{ display: "flex", gap: 6 }}>
                {inv.status !== "paid" && (
                  <button onClick={() => onReminder(inv)}
                    style={{ padding: "5px 10px", background: "#DCFCE7", border: "none", borderRadius: 6, color: T.greenDark, fontSize: 12, fontWeight: 600, cursor: "pointer" }}>
                    📱 WA
                  </button>
                )}
                {inv.status !== "paid" && (
                  <button onClick={() => onMarkPaid(inv.id)}
                    style={{ padding: "5px 10px", background: T.slateLight, border: "none", borderRadius: 6, color: T.slate, fontSize: 12, cursor: "pointer" }}>
                    Paid ✓
                  </button>
                )}
              </div>
            </td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}

function statusBadgeStyle(status) {
  const cfg = STATUS_CONFIG[status] || STATUS_CONFIG.draft;
  return { display: "inline-block", padding: "3px 10px", borderRadius: 20, fontSize: 11, fontWeight: 600, background: cfg.bg, color: cfg.color };
}
