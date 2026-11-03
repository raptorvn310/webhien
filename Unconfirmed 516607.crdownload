/* =====================================================================
   🌷 LỚP HỌC XINH XINH — Web quản lý học thêm
   Quản lý danh sách học sinh + điểm & nhận xét theo từng bài kiểm tra
   HTML/JS thuần + Firebase (Auth + Firestore)
   ===================================================================== */
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.13.2/firebase-app.js";
import { getAuth, signInWithEmailAndPassword, signOut, onAuthStateChanged }
  from "https://www.gstatic.com/firebasejs/10.13.2/firebase-auth.js";
import { getFirestore, collection, doc, getDoc, getDocs, addDoc, updateDoc, writeBatch, serverTimestamp }
  from "https://www.gstatic.com/firebasejs/10.13.2/firebase-firestore.js";

/* ======================= ⚙️ CẤU HÌNH — SỬA Ở ĐÂY ======================= */
const firebaseConfig = {
  apiKey: "AIzaSyCwDQIxoEWdxvgljaXmITxRrovBpQCyyxg",
  authDomain: "webhocthem-hien.firebaseapp.com",
  projectId: "webhocthem-hien",
  storageBucket: "webhocthem-hien.firebasestorage.app",
  messagingSenderId: "138972560088",
  appId: "1:138972560088:web:6010add0e93ff95cf8b5ad",
  measurementId: "G-T1C97E0XKK"
};
const TEN_WEB = "Lớp học cô Hiền";     // tên hiện trên web
const TEN_CO  = "Cô Hiền";             // lời chào, nút đăng nhập
/* ====================================================================== */

const CHUA_CAU_HINH = firebaseConfig.apiKey.startsWith("DAN_");
const fb = initializeApp(firebaseConfig);
const auth = getAuth(fb);
const db = getFirestore(fb);

/* ---------------------------- Tiện ích ---------------------------- */
const $ = s => document.querySelector(s);
const $$ = s => [...document.querySelectorAll(s)];
const val = s => ($(s)?.value ?? "");
const esc = s => String(s ?? "").replace(/[&<>"']/g, c => ({ "&": "&amp;", "<": "&lt;", ">": "&gt;", '"': "&quot;", "'": "&#39;" }[c]));
const boDau = s => String(s || "").normalize("NFD").replace(/[̀-ͯ]/g, "").replace(/đ/g, "d").replace(/Đ/g, "D");
const chuanTen = s => boDau(s).toLowerCase().replace(/\s+/g, " ").trim();
const gonTen = s => String(s || "").replace(/\s+/g, " ").trim();
const tenRieng = s => String(s || "").trim().split(/\s+/).pop() || "";
const soSanhTen = (a, b) => tenRieng(a).localeCompare(tenRieng(b), "vi") || String(a).localeCompare(String(b), "vi");
const soSanhSo = (a, b) => String(a ?? "").localeCompare(String(b ?? ""), "vi", { numeric: true });
const chuanSdt = s => { let d = String(s || "").replace(/\D/g, ""); if (d.startsWith("84") && d.length === 11) d = "0" + d.slice(2); if (d.length === 9 && d[0] !== "0") d = "0" + d; return d; };
const toDate = v => v?.toDate ? v.toDate() : (v ? new Date(v) : null);
const ms = v => toDate(v)?.getTime() || 0;
const pad = n => String(n).padStart(2, "0");
const thangNay = () => { const d = new Date(); return `${d.getFullYear()}-${pad(d.getMonth() + 1)}`; };
const tenThang = t => { if (!t) return ""; const [y, m] = String(t).split("-"); return `Tháng ${+m}/${y}`; };
const thangNgan = t => { if (!t) return ""; const [y, m] = String(t).split("-"); return `T${+m}/${String(y).slice(2)}`; };
const lamTron = v => Math.round(v * 100) / 100;
const fmtSo = v => (v === null || v === undefined || !isFinite(v)) ? "—" : String(lamTron(v)).replace(".", ",");
const docSo = s => { const t = String(s ?? "").trim().replace(",", "."); if (!/^\d+(\.\d+)?$/.test(t)) return null; const n = Number(t); return n >= 0 && n <= 10 ? n : null; };
const lopDiem = v => (v === null || v === undefined) ? "d-n" : v >= 8 ? "d-a" : v >= 6.5 ? "d-b" : v >= 5 ? "d-c" : "d-d";
const badge = s => { const t = String(s ?? "").trim(); return `<span class="diem-badge ${lopDiem(docSo(t))}">${esc(t || "—")}</span>`; };
const MAU = ["#ff8fab", "#6fd1ad", "#86c4f2", "#b6a0f4", "#ffb07c", "#f7a8d8", "#7cc9d6", "#e8b04a"];
const mauAv = s => { let h = 0; for (const c of String(s)) h = (h * 31 + c.charCodeAt(0)) >>> 0; return MAU[h % MAU.length]; };
const avatar = ten => `<div class="av" style="background:${mauAv(ten)}">${esc(tenRieng(ten).charAt(0).toUpperCase())}</div>`;
const empty = (icon, tieuDe, loi) => `<div class="empty"><span class="big">${icon}</span><b>${tieuDe}</b>${loi ? `<p>${loi}</p>` : ""}</div>`;

const MEO = `<svg class="meo" viewBox="0 0 120 110" aria-hidden="true">
<path d="M20 50 L25 10 L52 33 Z" fill="#fff" stroke="#6b5a70" stroke-width="3.5" stroke-linejoin="round"/>
<path d="M100 50 L95 10 L68 33 Z" fill="#fff" stroke="#6b5a70" stroke-width="3.5" stroke-linejoin="round"/>
<path d="M29 39 L30 22 L43 32 Z" fill="#ffc2d4"/><path d="M91 39 L90 22 L77 32 Z" fill="#ffc2d4"/>
<ellipse cx="60" cy="66" rx="46" ry="38" fill="#fff" stroke="#6b5a70" stroke-width="3.5"/>
<circle cx="42" cy="63" r="5.2" fill="#6b5a70"/><circle cx="78" cy="63" r="5.2" fill="#6b5a70"/>
<circle cx="43.8" cy="61.2" r="1.7" fill="#fff"/><circle cx="79.8" cy="61.2" r="1.7" fill="#fff"/>
<ellipse cx="29" cy="77" rx="8" ry="5" fill="#ffb3c8" opacity=".85"/><ellipse cx="91" cy="77" rx="8" ry="5" fill="#ffb3c8" opacity=".85"/>
<circle cx="60" cy="71" r="2.8" fill="#ff8fab"/>
<path d="M53 75 Q56.5 80 60 75 Q63.5 80 67 75" fill="none" stroke="#6b5a70" stroke-width="3" stroke-linecap="round"/>
<path d="M98 22 c-9-6-11-13-6-16 c3-2 6 0 6 3 c0-3 3-5 6-3 c5 3 3 10-6 16z" fill="#ff8fab"/>
</svg>`;

/* ------------------------- Modal & thông báo ------------------------- */
function moModal(tieuDe, body, { rong = false, foot = "" } = {}) {
  const el = document.createElement("div");
  el.className = "mb";
  el.innerHTML = `<div class="md ${rong ? "rong" : ""}">
    <div class="md-h"><h3>${tieuDe}</h3><button class="ic" onclick="A.dongModal()" aria-label="Đóng">✕</button></div>
    <div class="md-b">${body}</div>${foot ? `<div class="md-f">${foot}</div>` : ""}</div>`;
  $("#modalRoot").appendChild(el);
  document.body.classList.add("noscroll");
  return el;
}
function dongModal() {
  const r = $("#modalRoot");
  r.lastElementChild?.remove();
  if (!r.children.length) document.body.classList.remove("noscroll");
}
function hoi(msgHtml, ok = "Đồng ý", nguyHiem = false) {
  return new Promise(res => {
    A._tl = v => { dongModal(); res(v); };
    moModal("🤔 Xác nhận chút nha", `<p>${msgHtml}</p>`, {
      foot: `<button class="btn ghost" onclick="A._tl(false)">Thôi</button>
             <button class="btn ${nguyHiem ? "red" : ""}" onclick="A._tl(true)">${ok}</button>`
    });
  });
}
function toast(m, loai = "ok") {
  const t = document.createElement("div");
  t.className = "toast " + loai; t.textContent = m;
  $("#toastRoot").appendChild(t);
  setTimeout(() => t.classList.add("hide"), 2800);
  setTimeout(() => t.remove(), 3200);
}
const loiDep = e => {
  const c = e?.code || "";
  if (c.includes("invalid-credential") || c.includes("wrong-password") || c.includes("user-not-found")) return "Sai email hoặc mật khẩu rồi nè 🥲";
  if (c.includes("too-many-requests")) return "Thử sai nhiều quá, đợi một lát rồi thử lại nha";
  if (c.includes("permission-denied")) return "Không có quyền — kiểm tra lại email cô giáo trong Firestore Rules";
  if (c.includes("network") || c.includes("unavailable")) return "Mạng đang chập chờn, thử lại nha";
  return "Có lỗi: " + (e?.message || e);
};
const thu = fn => async (...a) => { try { return await fn(...a); } catch (e) { console.error(e); toast(loiDep(e), "err"); } };
function chonFile(accept) {
  return new Promise(res => {
    const i = document.createElement("input");
    i.type = "file"; i.accept = accept;
    i.onchange = () => res(i.files[0] || null);
    i.click();
  });
}
async function docExcel(file) {
  const wb = XLSX.read(await file.arrayBuffer());
  const ws = wb.Sheets[wb.SheetNames[0]];
  const rows = XLSX.utils.sheet_to_json(ws, { defval: "", raw: false });
  return rows.map(r => { const o = {}; for (const k in r) o[chuanTen(k).replace(/[^a-z0-9]/g, "")] = String(r[k] ?? "").trim(); return o; });
}
const lay = (row, ...keys) => { for (const k of keys) if (row[k] !== undefined && row[k] !== "") return row[k]; return ""; };
function xuatExcel(tenFile, aoa, rong = []) {
  const ws = XLSX.utils.aoa_to_sheet(aoa);
  if (rong.length) ws["!cols"] = rong.map(w => ({ wch: w }));
  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, ws, "Sheet1");
  XLSX.writeFile(wb, tenFile.replace(/[\\/:*?"<>|]/g, "-"));
}
async function chayBatch(ops) {
  for (let i = 0; i < ops.length; i += 400) {
    const b = writeBatch(db);
    ops.slice(i, i + 400).forEach(f => f(b));
    await b.commit();
  }
}
async function copyChu(text, tieuDe) {
  try { await navigator.clipboard.writeText(text); toast("Đã copy link 📋 — dán vào Zalo/tin nhắn gửi phụ huynh nha"); }
  catch { moModal(tieuDe, `<p>Copy link dưới đây gửi phụ huynh nha:</p><input value="${esc(text)}" onclick="this.select()" readonly>`); }
}
/* Dán nhiều ô từ Excel vào bảng nhập: trả về lưới 2 chiều hoặc null nếu chỉ là 1 ô */
function docLuoiDan(e) {
  const text = (e.clipboardData || window.clipboardData)?.getData("text") || "";
  if (!/[\t\n]/.test(text.replace(/\r?\n$/, ""))) return null;
  e.preventDefault();
  const dong = text.replace(/\r/g, "").split("\n");
  while (dong.length && dong[dong.length - 1] === "") dong.pop();
  return dong.map(d => d.split("\t"));
}

/* ---------------------------- Trạng thái ---------------------------- */
const S = {
  view: "loading0",
  namHoc: [], lop: [], hs: [], bai: [],
  sdtCu: new Set(), ckCu: new Set(),
  nhChon: "", tab: "hs", lopChon: "",
  hsSua: false, hsNhap: [], hsTick: new Set(),
  dLop: "", dThang: "", baiMo: "", bdSua: false,
  htLop: "", htHs: ""
};
const A = {};
window.A = A;
A.dongModal = dongModal;

const lopCuaNam = () => S.lop.filter(l => l.namHocId === S.nhChon).sort((a, b) => soSanhSo(a.ten, b.ten));
const tenLop = id => S.lop.find(l => l.id === id)?.ten || "";
const hsCuaLop = id => S.hs.filter(h => h.lopId === id).sort((a, b) => soSanhTen(a.hoTen, b.hoTen));
const baiCuaLop = id => S.bai.filter(b => b.lopId === id).sort((a, b) => String(b.thang).localeCompare(String(a.thang)) || ms(b.taoLuc) - ms(a.taoLuc));
const diemCua = (b, hsId) => b.diem?.[hsId] || {};

async function taiDuLieu() {
  const ten = ["namHoc", "lop", "hocSinh", "baiKiemTra", "bangDiemTheoSdt", "congKhaiDiem"];
  const snaps = await Promise.all(ten.map(n => getDocs(collection(db, n))));
  const m = s => s.docs.map(x => ({ id: x.id, ...x.data() }));
  [S.namHoc, S.lop, S.hs, S.bai] = snaps.slice(0, 4).map(m);
  S.sdtCu = new Set(snaps[4].docs.map(x => x.id));
  S.ckCu = new Set(snaps[5].docs.map(x => x.id));
  S.namHoc.sort((a, b) => soSanhSo(b.ten, a.ten));
  if (!S.namHoc.find(n => n.id === S.nhChon)) S.nhChon = S.namHoc[0]?.id || "";
}

/* Đồng bộ bản sao công khai cho phụ huynh (chỉ họ tên, SĐT, điểm, nhận xét) */
async function dongBoPhuHuynh() {
  const ops = [];
  const theoSdt = new Map();
  const tenLopDay = l => { const nh = S.namHoc.find(n => n.id === l.namHocId); return l.ten + (nh ? ` (${nh.ten})` : ""); };
  for (const h of S.hs) {
    const sdt = chuanSdt(h.sdtPH); const l = S.lop.find(x => x.id === h.lopId);
    if (!sdt || !l) continue;
    const ds = S.bai.filter(b => b.lopId === l.id)
      .sort((a, b) => String(a.thang).localeCompare(String(b.thang)) || ms(a.taoLuc) - ms(b.taoLuc))
      .map(b => ({ thang: b.thang, tenBai: b.ten, diem: diemCua(b, h.id).d || "", nhanXet: diemCua(b, h.id).nx || "" }))
      .filter(x => x.diem || x.nhanXet);
    if (!theoSdt.has(sdt)) theoSdt.set(sdt, []);
    theoSdt.get(sdt).push({ hoTen: h.hoTen, tenLop: tenLopDay(l), baiKiemTra: ds });
  }
  for (const [sdt, hocSinh] of theoSdt) ops.push(b => b.set(doc(db, "bangDiemTheoSdt", sdt), { hocSinh, capNhat: serverTimestamp() }));
  for (const sdt of S.sdtCu) if (!theoSdt.has(sdt)) ops.push(b => b.delete(doc(db, "bangDiemTheoSdt", sdt)));
  const baiIds = new Set();
  for (const bai of S.bai) {
    const l = S.lop.find(x => x.id === bai.lopId); if (!l) continue;
    baiIds.add(bai.id);
    const hocSinh = hsCuaLop(l.id).map(h => ({ hoTen: h.hoTen, sdt: chuanSdt(h.sdtPH), diem: diemCua(bai, h.id).d || "", nhanXet: diemCua(bai, h.id).nx || "" }));
    ops.push(b => b.set(doc(db, "congKhaiDiem", bai.id), { tenBai: bai.ten, tenLop: tenLopDay(l), thang: bai.thang, hocSinh, capNhat: serverTimestamp() }));
  }
  for (const id of S.ckCu) if (!baiIds.has(id)) ops.push(b => b.delete(doc(db, "congKhaiDiem", id)));
  await chayBatch(ops);
  S.sdtCu = new Set(theoSdt.keys()); S.ckCu = baiIds;
}
async function luuRoiDongBo() {
  try { await dongBoPhuHuynh(); }
  catch (e) { console.error(e); toast("Đã lưu, nhưng chưa cập nhật được cổng phụ huynh: " + loiDep(e), "err"); }
}

/* ---------------------------- Điều hướng ---------------------------- */
const thamSo = new URLSearchParams(location.search);
const BAI_LINK = thamSo.get("bai");
const linkGoc = () => location.origin + location.pathname;

function render() {
  const app = $("#app"), v = S.view;
  if (v === "loading0" || v === "loading") app.innerHTML = `<div class="loading"><div class="bounce">🌷</div><p>Đang mở lớp học...</p></div>`;
  else if (v === "home") app.innerHTML = vHome();
  else if (v === "coLogin") app.innerHTML = vCoLogin();
  else if (v === "ph") app.innerHTML = vPH();
  else if (v === "xemBai") app.innerHTML = vXemBai();
  else if (v === "co") app.innerHTML = vCo();
}
A.di = v => { S.view = v; render(); window.scrollTo(0, 0); };

onAuthStateChanged(auth, async u => {
  if (BAI_LINK) { if (S.view === "loading0") { S.view = "xemBai"; render(); A.taiBaiLink(); } return; }
  if (u) {
    if (S.view === "co") return;
    S.view = "loading"; render();
    try { await taiDuLieu(); } catch (e) { toast(loiDep(e), "err"); }
    S.view = "co"; render();
  } else if (["loading0", "loading", "co"].includes(S.view)) { S.view = "home"; render(); }
});

/* =============================== TRANG CHỦ =============================== */
function vHome() {
  return `<div class="home">
    <div class="home-side"><i class="fa fa-heart deco d1"></i><i class="fa fa-star deco d2"></i><i class="fa fa-pencil deco d3"></i>
      ${MEO}<h1>${esc(TEN_WEB)}</h1><p>Học vui mỗi ngày, giỏi hơn mỗi tuần 🌷</p></div>
    <div class="home-main">
      ${CHUA_CAU_HINH ? `<div class="setup-warn">⚠️ <b>Chưa dán cấu hình Firebase.</b> Mở file <code>app.js</code>, dán đoạn <code>firebaseConfig</code> vào đầu file (xem file hướng dẫn) rồi tải lại trang nha.</div>` : ""}
      <h2>Xin chào! 👋</h2><p>Chọn đúng vai của mình để vào nha.</p>
      <div class="home-cards">
        <button class="hcard" onclick="A.di('coLogin')"><span class="hi"><i class="fa fa-graduation-cap"></i></span><div><b>${esc(TEN_CO)}</b><small>Đăng nhập quản lý lớp, nhập điểm</small></div><i class="fa fa-angle-right"></i></button>
        <button class="hcard" onclick="A.di('ph')"><span class="hi"><i class="fa fa-heart"></i></span><div><b>Phụ huynh</b><small>Xem điểm & nhận xét của con</small></div><i class="fa fa-angle-right"></i></button>
      </div></div></div>`;
}
const khungAuth = (tieuDe, inner, coVe = true) => `<div class="auth-wrap"><div class="auth">
  ${coVe ? `<button class="btn ghost sm back" onclick="A.di('home')">← Về</button>` : ""}
  ${MEO}<h2>${tieuDe}</h2>${inner}</div></div>`;
function vCoLogin() {
  return khungAuth("👩‍🏫 Cô giáo đăng nhập", `
    <label>Email<input id="em" type="email" autocomplete="username" placeholder="email@gmail.com"></label>
    <label>Mật khẩu<input id="mk" type="password" autocomplete="current-password" onkeydown="if(event.key==='Enter')A.coDangNhap()"></label>
    <button class="btn full" id="btnDN" onclick="A.coDangNhap()">Đăng nhập 💖</button>`);
}
A.coDangNhap = async () => {
  const b = $("#btnDN"); b.disabled = true;
  try { await signInWithEmailAndPassword(auth, val("#em").trim(), val("#mk")); }
  catch (e) { toast(loiDep(e), "err"); b.disabled = false; }
};
A.dangXuat = async () => { await signOut(auth); S.view = "home"; render(); };

/* ============================ KHUNG CÔ GIÁO ============================ */
const TABS = [["hs", "fa-users", "Học sinh & Lớp"], ["diem", "fa-pencil-square-o", "Điểm kiểm tra"], ["ht", "fa-line-chart", "Học tập từng em"]];
function vCo() {
  const nhSel = `<select class="nh-sel" onchange="A.chonNamHoc(this.value)" title="Năm học">
      ${S.namHoc.length ? S.namHoc.map(n => `<option value="${n.id}" ${n.id === S.nhChon ? "selected" : ""}>${esc(n.ten)}</option>`).join("") : `<option>Chưa có năm học</option>`}</select>`;
  return `<aside id="sidebar">
      <div class="logo">${MEO}<b>${esc(TEN_WEB)}</b><small>Chào ${esc(TEN_CO)} 🌸</small></div>
      <div class="side-nh"><label>Năm học</label><div class="side-nh-row">${nhSel}<button class="ic" title="Thêm / sửa năm học" onclick="A.formNamHoc()"><i class="fa fa-calendar-plus-o"></i></button></div></div>
      <nav class="sticky-menu"><ul>${TABS.map(([k, ic, t]) => `<li class="${S.tab === k ? "current" : ""}"><a href="#" onclick="A.tab('${k}');return false"><i class="fa ${ic} ti"></i>${t}<i class="fa fa-angle-left"></i></a></li>`).join("")}</ul></nav>
      <div class="side-foot">
        <button onclick="A.taiLai()"><i class="fa fa-refresh"></i> Tải lại dữ liệu</button>
        <button onclick="A.dangXuat()"><i class="fa fa-sign-out"></i> Đăng xuất</button>
        <div class="copy">♥ ${esc(TEN_WEB)}</div></div>
    </aside>
    <header class="top-m">${MEO}${nhSel}
      <button class="ic" onclick="A.formNamHoc()"><i class="fa fa-calendar-plus-o"></i></button>
      <button class="ic" onclick="A.taiLai()"><i class="fa fa-refresh"></i></button>
      <button class="ic" onclick="A.dangXuat()"><i class="fa fa-sign-out"></i></button></header>
    <div id="content-section"><main id="tabMain">${vTab()}</main></div>
    <nav class="bottom-m">${TABS.map(([k, ic, t]) => `<button class="${S.tab === k ? "on" : ""}" onclick="A.tab('${k}')"><i class="fa ${ic}"></i>${t}</button>`).join("")}</nav>`;
}
function vTab() {
  if (!S.namHoc.length) return `<section class="panel"><div class="panel-h"><h2><i class="fa fa-calendar"></i>Năm học</h2></div>${empty("📅", "Bắt đầu bằng việc tạo năm học nha", "Ví dụ: 2026 – 2027. Sau đó tạo lớp và thêm học sinh.")}
    <div style="text-align:center"><button class="btn" onclick="A.formNamHoc()">＋ Tạo năm học</button></div></section>`;
  if (S.tab === "hs") return vTabHS();
  if (S.tab === "diem") return vTabDiem();
  return vTabHT();
}
const veTab = () => { if ($("#tabMain")) $("#tabMain").innerHTML = vTab(); };
A.tab = async t => {
  if (S.hsSua && t !== "hs" && !await hoi("Bảng học sinh đang mở để nhập mà chưa lưu. Bỏ các thay đổi chưa lưu nha?", "Bỏ thay đổi", true)) return;
  if (S.bdSua && t !== "diem" && !await hoi("Bảng điểm đang mở để nhập mà chưa lưu. Bỏ các thay đổi chưa lưu nha?", "Bỏ thay đổi", true)) return;
  S.hsSua = false; S.bdSua = false;
  S.tab = t; render(); window.scrollTo(0, 0);
};
A.chonNamHoc = id => { S.nhChon = id; S.lopChon = S.dLop = S.htLop = ""; S.baiMo = ""; S.hsSua = S.bdSua = false; render(); };
A.taiLai = thu(async () => { await taiDuLieu(); S.hsSua = S.bdSua = false; render(); toast("Đã cập nhật dữ liệu mới nhất ✨"); });

A.formNamHoc = () => {
  const nh = S.namHoc.find(n => n.id === S.nhChon);
  const y = new Date().getFullYear(), goiY = new Date().getMonth() >= 6 ? `${y} – ${y + 1}` : `${y - 1} – ${y}`;
  moModal("📅 Năm học", `
    <label>Tạo năm học mới<input id="nh_moi" placeholder="${goiY}" value="${S.namHoc.some(n => n.ten === goiY) ? "" : goiY}"></label>
    ${nh ? `<hr style="border:none;border-top:2px dashed var(--line);margin:14px 0">
    <label>Đổi tên năm học đang chọn<input id="nh_ten" value="${esc(nh.ten)}"></label>` : ""}`,
    { foot: `<button class="btn ghost" onclick="A.dongModal()">Đóng</button>${nh ? `<button class="btn ghost danger" onclick="A.xoaNamHoc()">🗑️ Xóa năm đang chọn</button><button class="btn sky" onclick="A.doiTenNamHoc()">Lưu tên</button>` : ""}<button class="btn" onclick="A.taoNamHoc()">＋ Tạo mới</button>` });
};
A.taoNamHoc = thu(async () => {
  const ten = val("#nh_moi").trim(); if (!ten) return toast("Nhập tên năm học nha", "err");
  const r = await addDoc(collection(db, "namHoc"), { ten, taoLuc: serverTimestamp() });
  S.namHoc.push({ id: r.id, ten }); S.namHoc.sort((a, b) => soSanhSo(b.ten, a.ten));
  dongModal(); A.chonNamHoc(r.id); toast("Đã tạo năm học 🎉");
});
A.doiTenNamHoc = thu(async () => {
  const ten = val("#nh_ten").trim(); if (!ten) return;
  await updateDoc(doc(db, "namHoc", S.nhChon), { ten });
  S.namHoc.find(n => n.id === S.nhChon).ten = ten; dongModal(); render(); toast("Đã đổi tên"); luuRoiDongBo();
});
A.xoaNamHoc = thu(async () => {
  if (lopCuaNam().length) return toast("Năm học này còn lớp — xóa hết lớp trước nha", "err");
  if (!await hoi("Xóa năm học đang chọn nha?", "Xóa", true)) return;
  await chayBatch([b => b.delete(doc(db, "namHoc", S.nhChon))]);
  S.namHoc = S.namHoc.filter(n => n.id !== S.nhChon); dongModal(); A.chonNamHoc(S.namHoc[0]?.id || "");
});

/* ============================ TAB 1: HỌC SINH & LỚP ============================ */
const COT = [["lopGhiChu", "Lớp", 90], ["hoTen", "Họ tên", 0], ["sdtPH", "SĐT phụ huynh", 130], ["hocBatDau", "Học bắt đầu", 120]];
function vTabHS() {
  const ds = lopCuaNam();
  if (!ds.find(l => l.id === S.lopChon)) S.lopChon = ds[0]?.id || "";
  const lop = ds.find(l => l.id === S.lopChon);
  const chips = ds.map(l => `<button class="chip ${l.id === S.lopChon ? "on" : ""}" onclick="A.chonLop('${l.id}')">${esc(l.ten)} <span class="cnt">${S.hs.filter(h => h.lopId === l.id).length}</span></button>`).join("");
  let body = "";
  if (!ds.length) body = empty("🐣", "Năm học này chưa có lớp nào", "Bấm “＋ Lớp mới”, hoặc “Nhập từ Excel (nhiều lớp)” để tạo nhanh nha.");
  else if (lop) {
    const hs = hsCuaLop(lop.id);
    body = `<div class="lop-head"><h3>🏫 ${esc(lop.ten)} <span class="pill">${hs.length} em</span></h3>
        <span class="lop-act"><button class="btn sm ghost" onclick="A.formLop('${lop.id}')">✏️ Đổi tên</button><button class="btn sm ghost danger" onclick="A.xoaLop('${lop.id}')">🗑️ Xóa lớp</button></span></div>
      <div class="toolbar">
        ${S.hsSua
          ? `<button class="btn" onclick="A.luuHS()">🔒 Khóa & Lưu</button><button class="btn ghost" onclick="A.themDong()">＋ Thêm dòng trống</button><button class="btn ghost" onclick="A.huyHS()">Hủy</button>`
          : `<button class="btn peach" onclick="A.moNhapHS()">✏️ Nhập</button>
             <button class="btn mint" onclick="A.capNhatExcel()">📥 Cập nhật từ Excel</button>
             <button class="btn sky" onclick="A.xuatLop()">📤 Xuất Excel lớp này</button>
             <button class="btn ghost danger" onclick="A.xoaTick()" ${S.hsTick.size ? "" : "disabled"}>🗑️ Xóa đã chọn (${S.hsTick.size})</button>`}
      </div>
      <p class="hint">${S.hsSua
        ? "💡 Đang mở để nhập: gõ trực tiếp, hoặc copy nhiều ô trong Excel rồi bấm vào ô đầu tiên và dán (Ctrl+V) — dán dư dòng sẽ tự thêm dòng. Dòng xóa trắng sẽ bị xóa khỏi danh sách khi lưu."
        : "💡 Bấm “✏️ Nhập” để thêm/sửa học sinh (dán thẳng từ Excel được). Cột <b>Lớp</b> chỉ là ghi chú riêng của từng em (vd lớp trên trường), không làm em đổi sang lớp khác."}</p>
      <div class="tbl-wrap"><table class="tbl sheet" id="hsTable">
        <tr>${S.hsSua ? "" : `<th style="width:34px"><input type="checkbox" onchange="A.tickHet(this.checked)" ${hs.length && S.hsTick.size === hs.length ? "checked" : ""}></th>`}<th style="width:46px">STT</th>${COT.map(([, t, w]) => `<th ${w ? `style="width:${w}px"` : ""}>${t}</th>`).join("")}</tr>
        ${S.hsSua ? vDongNhap() : hs.map((h, i) => `<tr class="${S.hsTick.has(h.id) ? "tick" : ""}"><td><input type="checkbox" ${S.hsTick.has(h.id) ? "checked" : ""} onchange="A.tick('${h.id}',this.checked)"></td><td>${i + 1}</td>
          <td>${esc(h.lopGhiChu)}</td><td><b>${esc(h.hoTen)}</b></td><td>${esc(h.sdtPH)}</td><td>${esc(h.hocBatDau)}</td></tr>`).join("")}
      </table></div>
      ${!S.hsSua && !hs.length ? empty("🌱", "Lớp chưa có học sinh", "Bấm “✏️ Nhập” rồi gõ hoặc dán danh sách từ Excel nha.") : ""}`;
  }
  return `<section class="panel"><div class="panel-h"><h2><i class="fa fa-users"></i>Học sinh & Lớp</h2>
      <button class="btn sm sky" onclick="A.xuatToanBo()">📤 Xuất Excel toàn bộ</button>
      <button class="btn sm mint" onclick="A.nhapNhieuLop()">📥 Nhập từ Excel (nhiều lớp)</button></div>
    <div class="chips">${chips}<button class="chip add" onclick="A.formLop()">＋ Lớp mới</button></div>
    ${body}</section>`;
}
function vDongNhap() {
  return S.hsNhap.map((r, i) => `<tr><td>${i + 1}</td>${COT.map(([k], c) => `<td class="o"><input data-r="${i}" data-c="${c}" value="${esc(r[k])}" ${k === "sdtPH" ? 'inputmode="tel"' : ""}
    oninput="A.oNhap(this)" onpaste="A.danHS(event,this)"></td>`).join("")}</tr>`).join("");
}
const dongTrong = () => ({ id: "", lopGhiChu: "", hoTen: "", sdtPH: "", hocBatDau: "" });
A.chonLop = id => { if (S.hsSua) return toast("Bấm “Khóa & Lưu” hoặc “Hủy” trước khi đổi lớp nha", "err"); S.lopChon = id; S.hsTick.clear(); veTab(); };
A.tick = (id, c) => { c ? S.hsTick.add(id) : S.hsTick.delete(id); veTab(); };
A.tickHet = c => { S.hsTick.clear(); if (c) hsCuaLop(S.lopChon).forEach(h => S.hsTick.add(h.id)); veTab(); };
A.moNhapHS = () => {
  S.hsNhap = hsCuaLop(S.lopChon).map(h => ({ id: h.id, lopGhiChu: h.lopGhiChu || "", hoTen: h.hoTen || "", sdtPH: h.sdtPH || "", hocBatDau: h.hocBatDau || "" }));
  for (let i = 0; i < (S.hsNhap.length ? 3 : 10); i++) S.hsNhap.push(dongTrong());
  S.hsSua = true; S.hsTick.clear(); veTab();
};
A.huyHS = () => { S.hsSua = false; veTab(); };
A.themDong = () => { for (let i = 0; i < 5; i++) S.hsNhap.push(dongTrong()); veTab(); };
A.oNhap = el => { S.hsNhap[+el.dataset.r][COT[+el.dataset.c][0]] = el.value; };
A.danHS = (e, el) => {
  const luoi = docLuoiDan(e); if (!luoi) return;
  const r0 = +el.dataset.r, c0 = +el.dataset.c;
  luoi.forEach((dong, i) => {
    while (S.hsNhap.length <= r0 + i) S.hsNhap.push(dongTrong());
    dong.forEach((o, j) => { if (c0 + j < COT.length) S.hsNhap[r0 + i][COT[c0 + j][0]] = o.trim(); });
  });
  const y = window.scrollY; veTab(); window.scrollTo(0, y);
  toast(`Đã dán ${luoi.length} dòng — kiểm tra rồi bấm “Khóa & Lưu” nha`);
};
A.luuHS = thu(async () => {
  const lopId = S.lopChon, ops = [], them = [], sua = [], xoa = [];
  for (const [i, r] of S.hsNhap.entries()) {
    const d = { lopGhiChu: gonTen(r.lopGhiChu), hoTen: gonTen(r.hoTen), sdtPH: chuanSdt(r.sdtPH), hocBatDau: gonTen(r.hocBatDau) };
    const trong = !d.lopGhiChu && !d.hoTen && !d.sdtPH && !d.hocBatDau;
    if (trong) { if (r.id) xoa.push(r.id); continue; }
    if (!d.hoTen) return toast(`Dòng ${i + 1} chưa có họ tên nè`, "err");
    if (r.id) {
      const cu = S.hs.find(h => h.id === r.id);
      if (COT.some(([k]) => (cu[k] || "") !== d[k])) sua.push({ id: r.id, d });
    } else them.push({ ref: doc(collection(db, "hocSinh")), d: { ...d, lopId } });
  }
  sua.forEach(x => ops.push(b => b.update(doc(db, "hocSinh", x.id), x.d)));
  them.forEach(x => ops.push(b => b.set(x.ref, { ...x.d, taoLuc: serverTimestamp() })));
  xoa.forEach(id => ops.push(b => b.delete(doc(db, "hocSinh", id))));
  if (ops.length) await chayBatch(ops);
  sua.forEach(x => Object.assign(S.hs.find(h => h.id === x.id), x.d));
  them.forEach(x => S.hs.push({ id: x.ref.id, ...x.d }));
  S.hs = S.hs.filter(h => !xoa.includes(h.id));
  S.hsSua = false; veTab();
  toast(ops.length ? `Đã lưu 💖 (thêm ${them.length}, sửa ${sua.length}, xóa ${xoa.length})` : "Không có gì thay đổi 🌼");
  if (ops.length) luuRoiDongBo();
});
A.xoaTick = thu(async () => {
  const ids = [...S.hsTick]; if (!ids.length) return;
  if (!await hoi(`Xóa <b>${ids.length}</b> học sinh đã chọn khỏi lớp nha?`, "Xóa", true)) return;
  await chayBatch(ids.map(id => b => b.delete(doc(db, "hocSinh", id))));
  S.hs = S.hs.filter(h => !S.hsTick.has(h.id)); S.hsTick.clear(); veTab();
  toast("Đã xóa"); luuRoiDongBo();
});
A.formLop = (id = "") => {
  const l = S.lop.find(x => x.id === id) || {};
  moModal(id ? "✏️ Đổi tên lớp" : "🏫 Tạo lớp mới", `<label>Tên lớp<input id="l_ten" value="${esc(l.ten || "")}" placeholder="VD: Toán 7 – Tối 2,4,6" onkeydown="if(event.key==='Enter')A.luuLop('${id}')"></label>`,
    { foot: `<button class="btn ghost" onclick="A.dongModal()">Hủy</button><button class="btn" onclick="A.luuLop('${id}')">Lưu 💖</button>` });
  setTimeout(() => $("#l_ten")?.focus(), 50);
};
A.luuLop = thu(async id => {
  const ten = val("#l_ten").trim(); if (!ten) return toast("Nhập tên lớp nha", "err");
  if (id) { await updateDoc(doc(db, "lop", id), { ten }); S.lop.find(x => x.id === id).ten = ten; }
  else { const r = await addDoc(collection(db, "lop"), { ten, namHocId: S.nhChon, taoLuc: serverTimestamp() }); S.lop.push({ id: r.id, ten, namHocId: S.nhChon }); S.lopChon = r.id; }
  dongModal(); toast("Đã lưu lớp 🌷"); veTab();
  if (id) luuRoiDongBo();
});
A.xoaLop = thu(async id => {
  const hs = S.hs.filter(h => h.lopId === id), bai = S.bai.filter(b => b.lopId === id);
  if (!await hoi(`Xóa lớp <b>${esc(tenLop(id))}</b>${hs.length || bai.length ? ` cùng <b>${hs.length}</b> học sinh và <b>${bai.length}</b> bài kiểm tra của lớp` : ""} nha? Không khôi phục lại được đâu.`, "Xóa lớp", true)) return;
  await chayBatch([b => b.delete(doc(db, "lop", id)), ...hs.map(h => b => b.delete(doc(db, "hocSinh", h.id))), ...bai.map(x => b => b.delete(doc(db, "baiKiemTra", x.id)))]);
  S.lop = S.lop.filter(l => l.id !== id); S.hs = S.hs.filter(h => h.lopId !== id); S.bai = S.bai.filter(b => b.lopId !== id);
  toast("Đã xóa lớp"); veTab(); luuRoiDongBo();
});
const tuDongExcel = r => ({
  hoTen: gonTen(lay(r, "hovaten", "hoten", "hotenhocsinh", "ten")),
  sdtPH: chuanSdt(lay(r, "sdtphuhuynh", "sdtph", "sdt", "sodienthoai", "dienthoai")),
  lopGhiChu: gonTen(lay(r, "lop", "lophoctrentruong", "loptrentruong")),
  hocBatDau: gonTen(lay(r, "hocbatdau", "ngaybatdau", "batdau"))
});
A.nhapNhieuLop = thu(async () => {
  const f = await chonFile(".xlsx,.xls,.csv"); if (!f) return;
  const rows = (await docExcel(f)).map(tuDongExcel).filter(r => r.hoTen);
  if (!rows.length) return toast("Không đọc được học sinh nào — file cần cột “Họ và tên” và “Lớp”", "err");
  const ops = []; let them = 0, trung = 0, lopMoi = 0;
  const lopTheoTen = new Map(lopCuaNam().map(l => [chuanTen(l.ten), l]));
  for (const r of rows) {
    const tenL = r.lopGhiChu || "Chưa xếp lớp";
    let l = lopTheoTen.get(chuanTen(tenL));
    if (!l) {
      const ref = doc(collection(db, "lop"));
      l = { id: ref.id, ten: tenL, namHocId: S.nhChon };
      ops.push(b => b.set(ref, { ten: tenL, namHocId: S.nhChon, taoLuc: serverTimestamp() }));
      S.lop.push(l); lopTheoTen.set(chuanTen(tenL), l); lopMoi++;
    }
    if (S.hs.some(h => h.lopId === l.id && chuanTen(h.hoTen) === chuanTen(r.hoTen))) { trung++; continue; }
    const ref = doc(collection(db, "hocSinh"));
    const d = { ...r, lopId: l.id };
    ops.push(b => b.set(ref, { ...d, taoLuc: serverTimestamp() }));
    S.hs.push({ id: ref.id, ...d }); them++;
  }
  await chayBatch(ops);
  toast(`Đã thêm ${them} em${lopMoi ? `, tạo ${lopMoi} lớp mới` : ""}${trung ? `, bỏ qua ${trung} em đã có` : ""} 🎉`);
  veTab(); luuRoiDongBo();
});
A.capNhatExcel = thu(async () => {
  const f = await chonFile(".xlsx,.xls,.csv"); if (!f) return;
  const rows = (await docExcel(f)).map(tuDongExcel).filter(r => r.hoTen);
  if (!rows.length) return toast("Không đọc được học sinh nào — file cần cột “Họ và tên”", "err");
  const theTen = new Map(hsCuaLop(S.lopChon).map(h => [chuanTen(h.hoTen), h]));
  const ops = []; let sua = 0, them = 0;
  for (const r of rows) {
    const h = theTen.get(chuanTen(r.hoTen));
    const d = {}; for (const k of ["sdtPH", "lopGhiChu", "hocBatDau"]) if (r[k]) d[k] = r[k];
    if (h) { if (Object.keys(d).some(k => h[k] !== d[k])) { ops.push(b => b.update(doc(db, "hocSinh", h.id), d)); Object.assign(h, d); sua++; } }
    else {
      const ref = doc(collection(db, "hocSinh")), nd = { hoTen: r.hoTen, sdtPH: r.sdtPH, lopGhiChu: r.lopGhiChu, hocBatDau: r.hocBatDau, lopId: S.lopChon };
      ops.push(b => b.set(ref, { ...nd, taoLuc: serverTimestamp() })); S.hs.push({ id: ref.id, ...nd }); theTen.set(chuanTen(r.hoTen), nd); them++;
    }
  }
  if (ops.length) await chayBatch(ops);
  toast(`Cập nhật ${sua} em, thêm mới ${them} em ✨`); veTab(); if (ops.length) luuRoiDongBo();
});
const DIEM_DANH = Array.from({ length: 12 }, (_, i) => `Buổi ${i + 1}`);
const dongXuat = (h, i) => [i + 1, h.lopGhiChu || "", h.hoTen, h.sdtPH || "", h.hocBatDau || "", ...DIEM_DANH.map(() => "")];
A.xuatLop = () => {
  const l = S.lop.find(x => x.id === S.lopChon); if (!l) return;
  xuatExcel(`Danh sach ${l.ten}.xlsx`, [["STT", "Lớp", "Họ và tên", "SĐT phụ huynh", "Học bắt đầu", ...DIEM_DANH], ...hsCuaLop(l.id).map(dongXuat)], [5, 8, 26, 13, 12, ...DIEM_DANH.map(() => 7)]);
};
A.xuatToanBo = () => {
  const aoa = [["Lớp học thêm", "STT", "Lớp", "Họ và tên", "SĐT phụ huynh", "Học bắt đầu", ...DIEM_DANH]];
  lopCuaNam().forEach(l => hsCuaLop(l.id).forEach((h, i) => aoa.push([l.ten, ...dongXuat(h, i)])));
  if (aoa.length === 1) return toast("Chưa có học sinh nào để xuất", "err");
  xuatExcel(`Danh sach toan bo - ${S.namHoc.find(n => n.id === S.nhChon)?.ten || ""}.xlsx`, aoa, [16, 5, 8, 26, 13, 12, ...DIEM_DANH.map(() => 7)]);
};

/* ============================ TAB 2: ĐIỂM KIỂM TRA ============================ */
function vTabDiem() {
  const ds = lopCuaNam();
  if (!ds.find(l => l.id === S.dLop)) S.dLop = S.lopChon && ds.find(l => l.id === S.lopChon) ? S.lopChon : ds[0]?.id || "";
  const bais = baiCuaLop(S.dLop).filter(b => !S.dThang || b.thang === S.dThang);
  const bai = S.bai.find(b => b.id === S.baiMo && b.lopId === S.dLop);
  if (!bai) { S.baiMo = ""; S.bdSua = false; }
  const head = `<div class="panel-h"><h2><i class="fa fa-pencil-square-o"></i>Điểm kiểm tra</h2>
      <button class="btn sm lav" onclick="A.copyCong()">📋 Copy link cổng phụ huynh</button></div>
    <div class="sel-line">
      <label>Lớp<select onchange="A.dChon('dLop',this.value)">${ds.map(l => `<option value="${l.id}" ${l.id === S.dLop ? "selected" : ""}>${esc(l.ten)}</option>`).join("")}</select></label>
      <label>Tháng <small class="muted">(lọc, không bắt buộc)</small><span style="display:flex;gap:6px;margin-top:5px"><input type="month" value="${S.dThang}" onchange="A.dChon('dThang',this.value)" style="margin:0">${S.dThang ? `<button class="ic" title="Bỏ lọc" onclick="A.dChon('dThang','')">✕</button>` : ""}</span></label>
      <button class="btn" onclick="A.formBai()">＋ Thêm bài kiểm tra</button></div>
    <p class="hint">💌 Cổng phụ huynh là <b>1 link chung</b> cho tất cả phụ huynh — nhập SĐT là xem được toàn bộ điểm + nhận xét của con. Điểm vừa lưu sẽ tự cập nhật vào cổng.</p>`;
  if (!ds.length) return `<section class="panel">${head.replace(/<div class="sel-line">[\s\S]*<\/div>\s*<p/, "<p")}${empty("🐣", "Năm học này chưa có lớp nào", "Tạo lớp ở tab Học sinh & Lớp trước nha.")}</section>`;
  return `<section class="panel">${head}
    ${bais.length ? `<div class="grid-bai">${bais.map(b => {
      const hs = hsCuaLop(b.lopId), co = hs.filter(h => diemCua(b, h.id).d).length;
      const so = hs.map(h => docSo(diemCua(b, h.id).d)).filter(v => v !== null);
      return `<div class="bai-card ${b.id === S.baiMo ? "on" : ""}">
        <div class="bai-top"><span class="badge sky">${tenThang(b.thang)}</span><span class="badge ${co === hs.length && co ? "mo" : "peach"}">📝 ${co}/${hs.length} em có điểm</span></div>
        <h3>${esc(b.ten)}</h3>
        <small class="muted">TB lớp: <b>${so.length ? fmtSo(so.reduce((a, c) => a + c, 0) / so.length) : "—"}</b></small>
        <div class="de-act"><button class="btn sm" onclick="A.moBai('${b.id}')">👀 Xem điểm</button>
          <button class="btn sm ghost" onclick="A.copyBai('${b.id}')">🔗 Link</button>
          <button class="btn sm ghost" onclick="A.formBai('${b.id}')">✏️</button>
          <button class="btn sm ghost danger" onclick="A.xoaBai('${b.id}')">🗑️</button></div></div>`;
    }).join("")}</div>`
      : empty("📝", S.dThang ? `Chưa có bài kiểm tra nào trong ${tenThang(S.dThang).toLowerCase()}` : "Lớp này chưa có bài kiểm tra nào", "Bấm “＋ Thêm bài kiểm tra” để tạo bài, rồi nhập điểm + nhận xét cho từng em nha.")}
    ${bai ? vBangDiem(bai) : ""}</section>`;
}
function vBangDiem(b) {
  const hs = hsCuaLop(b.lopId);
  return `<div class="bang-diem" id="bangDiem">
    <div class="lop-head"><h3>📝 ${esc(b.ten)} <span class="badge sky">${tenThang(b.thang)}</span></h3>
      <span class="lop-act"><button class="ic" title="Đóng" onclick="A.dongBai()">✕</button></span></div>
    <div class="toolbar">
      ${S.bdSua ? `<button class="btn" onclick="A.luuDiem()">🔒 Khóa & Lưu</button><button class="btn ghost" onclick="A.huyDiem()">Hủy</button>`
        : `<button class="btn peach" onclick="A.moNhapDiem()">✏️ Nhập</button>
           <button class="btn mint" onclick="A.nhapDiemExcel()">📥 Nhập điểm từ Excel</button>
           <button class="btn sky" onclick="A.xuatBaoCao()">📤 Xuất báo cáo Excel</button>
           <button class="btn lav" onclick="A.copyBai('${b.id}')">🔗 Copy link cho phụ huynh</button>`}
    </div>
    <p class="hint">${S.bdSua ? "💡 Gõ điểm + nhận xét, hoặc copy 2 cột (Điểm, Nhận xét) trong Excel rồi dán vào ô đầu tiên. Ô điểm ghi chữ cũng được (vd “Vắng”)."
      : "💡 Bấm “✏️ Nhập” để gõ tay, hoặc “📥 Nhập điểm từ Excel” (cột: Họ tên, Điểm, Nhận xét — tự khớp theo họ tên)."}</p>
    ${hs.length ? `<div class="tbl-wrap"><table class="tbl sheet"><tr><th style="width:46px">STT</th><th>Họ tên</th><th style="width:110px" class="c">Điểm</th><th>Nhận xét</th></tr>
      ${hs.map((h, i) => { const d = diemCua(b, h.id); return `<tr data-hs="${h.id}"><td>${i + 1}</td><td><b>${esc(h.hoTen)}</b></td>
        ${S.bdSua ? `<td class="o"><input class="bd-d" data-r="${i}" data-c="0" value="${esc(d.d || "")}" inputmode="decimal" onpaste="A.danDiem(event,this)" style="text-align:center"></td>
          <td class="o"><textarea class="bd-nx" data-r="${i}" data-c="1" rows="1" onpaste="A.danDiem(event,this)">${esc(d.nx || "")}</textarea></td>`
        : `<td class="c">${badge(d.d)}</td><td class="nx">${esc(d.nx || "")}</td>`}</tr>`; }).join("")}
    </table></div>` : empty("🌱", "Lớp chưa có học sinh")}</div>`;
}
A.dChon = (k, v) => { if (S.bdSua) return toast("Bấm “Khóa & Lưu” hoặc “Hủy” bảng điểm trước nha", "err"); S[k] = v; if (k === "dLop") S.baiMo = ""; veTab(); };
A.moBai = id => {
  if (S.bdSua) return toast("Bấm “Khóa & Lưu” hoặc “Hủy” bảng điểm trước nha", "err");
  S.baiMo = id; veTab(); setTimeout(() => $("#bangDiem")?.scrollIntoView({ behavior: "smooth", block: "start" }), 50);
};
A.dongBai = () => { if (S.bdSua) return toast("Bấm “Khóa & Lưu” hoặc “Hủy” trước nha", "err"); S.baiMo = ""; veTab(); };
A.formBai = (id = "") => {
  const b = S.bai.find(x => x.id === id) || { thang: S.dThang || thangNay() };
  const thuTu = baiCuaLop(S.dLop).filter(x => x.thang === b.thang).length + 1;
  moModal(id ? "✏️ Sửa bài kiểm tra" : "📝 Thêm bài kiểm tra", `
    <label>Tên bài<input id="b_ten" value="${esc(b.ten || `Kiểm tra ${tenThang(b.thang).toLowerCase()} – lần ${thuTu}`)}"></label>
    <label>Tháng<input id="b_thang" type="month" value="${b.thang}"></label>
    <p class="muted" style="font-size:13px;margin:0">Lớp: <b>${esc(tenLop(id ? b.lopId : S.dLop))}</b></p>`,
    { foot: `<button class="btn ghost" onclick="A.dongModal()">Hủy</button><button class="btn" onclick="A.luuBai('${id}')">Lưu 💖</button>` });
  setTimeout(() => $("#b_ten")?.select(), 50);
};
A.luuBai = thu(async id => {
  const ten = val("#b_ten").trim(), thang = val("#b_thang");
  if (!ten) return toast("Nhập tên bài nha", "err");
  if (!thang) return toast("Chọn tháng nha", "err");
  if (id) { await updateDoc(doc(db, "baiKiemTra", id), { ten, thang }); Object.assign(S.bai.find(x => x.id === id), { ten, thang }); }
  else {
    const r = await addDoc(collection(db, "baiKiemTra"), { ten, thang, lopId: S.dLop, diem: {}, taoLuc: serverTimestamp() });
    S.bai.push({ id: r.id, ten, thang, lopId: S.dLop, diem: {}, taoLuc: new Date() }); S.baiMo = r.id;
  }
  dongModal(); toast("Đã lưu bài kiểm tra 🌷"); veTab(); luuRoiDongBo();
  if (!id) setTimeout(() => $("#bangDiem")?.scrollIntoView({ behavior: "smooth", block: "start" }), 50);
});
A.xoaBai = thu(async id => {
  const b = S.bai.find(x => x.id === id);
  if (!await hoi(`Xóa bài <b>${esc(b.ten)}</b> cùng toàn bộ điểm + nhận xét của bài này nha?`, "Xóa", true)) return;
  await chayBatch([x => x.delete(doc(db, "baiKiemTra", id))]);
  S.bai = S.bai.filter(x => x.id !== id); if (S.baiMo === id) { S.baiMo = ""; S.bdSua = false; }
  toast("Đã xóa bài"); veTab(); luuRoiDongBo();
});
A.moNhapDiem = () => { S.bdSua = true; veTab(); $("#bangDiem")?.scrollIntoView({ block: "start" }); };
A.huyDiem = () => { S.bdSua = false; veTab(); };
A.danDiem = (e, el) => {
  const luoi = docLuoiDan(e); if (!luoi) return;
  const r0 = +el.dataset.r, c0 = +el.dataset.c;
  const oD = $$(".bd-d"), oN = $$(".bd-nx");
  luoi.forEach((dong, i) => dong.forEach((o, j) => {
    const c = c0 + j, r = r0 + i;
    if (c === 0 && oD[r]) oD[r].value = o.trim();
    if (c === 1 && oN[r]) oN[r].value = o.trim();
  }));
  toast(`Đã dán ${luoi.length} dòng — kiểm tra rồi bấm “Khóa & Lưu” nha`);
};
async function luuDiemBai(b, moi) { // moi: {hsId:{d,nx}}
  await updateDoc(doc(db, "baiKiemTra", b.id), { diem: moi, capNhat: serverTimestamp() });
  b.diem = moi;
}
A.luuDiem = thu(async () => {
  const b = S.bai.find(x => x.id === S.baiMo); if (!b) return;
  const moi = { ...(b.diem || {}) };
  $$("#bangDiem tr[data-hs]").forEach(tr => {
    const d = tr.querySelector(".bd-d").value.trim(), nx = tr.querySelector(".bd-nx").value.trim();
    if (d || nx) moi[tr.dataset.hs] = { d, nx }; else delete moi[tr.dataset.hs];
  });
  await luuDiemBai(b, moi);
  S.bdSua = false; veTab(); toast("Đã lưu điểm 💖"); luuRoiDongBo();
});
A.nhapDiemExcel = thu(async () => {
  const b = S.bai.find(x => x.id === S.baiMo); if (!b) return;
  const f = await chonFile(".xlsx,.xls,.csv"); if (!f) return;
  const rows = await docExcel(f);
  const hs = hsCuaLop(b.lopId);
  const theTen = new Map(hs.map(h => [chuanTen(h.hoTen), h]));
  const moi = { ...(b.diem || {}) }; let n = 0; const khongKhop = [];
  for (const r of rows) {
    const ten = lay(r, "hoten", "hovaten", "ten"), sbd = lay(r, "sbd", "stt", "sobaodanh");
    let h = ten ? theTen.get(chuanTen(ten)) : null;
    if (!h && !ten && sbd && /^\d+$/.test(sbd)) h = hs[+sbd - 1];
    if (!h) { if (ten || sbd) khongKhop.push(ten || `SBD ${sbd}`); continue; }
    const d = lay(r, "diem", "diemso", "tongdiem"), nx = lay(r, "nhanxet", "ghichu", "loiphe");
    if (!d && !nx) continue;
    moi[h.id] = { d: d || moi[h.id]?.d || "", nx: nx || moi[h.id]?.nx || "" }; n++;
  }
  if (!n) return toast("Không khớp được em nào — file cần cột Họ tên (hoặc SBD), Điểm, Nhận xét", "err");
  await luuDiemBai(b, moi);
  veTab(); toast(`Đã nhập điểm cho ${n} em 🎉`); luuRoiDongBo();
  if (khongKhop.length) moModal("🧐 Các tên này không có trong lớp", `<p>${khongKhop.map(esc).join("<br>")}</p><p class="muted">Kiểm tra lại chính tả họ tên trong file rồi nhập lại nha (các em đã khớp vẫn được lưu).</p>`);
});
A.xuatBaoCao = () => {
  const b = S.bai.find(x => x.id === S.baiMo); if (!b) return;
  const aoa = [[`${b.ten} — Lớp ${tenLop(b.lopId)} — ${tenThang(b.thang)}`], [], ["STT", "Họ và tên", "SĐT phụ huynh", "Điểm", "Nhận xét"]];
  hsCuaLop(b.lopId).forEach((h, i) => aoa.push([i + 1, h.hoTen, h.sdtPH || "", diemCua(b, h.id).d || "", diemCua(b, h.id).nx || ""]));
  xuatExcel(`Bao cao ${b.ten} - ${tenLop(b.lopId)}.xlsx`, aoa, [5, 26, 13, 8, 60]);
};
A.copyBai = id => copyChu(`${linkGoc()}?bai=${id}`, "🔗 Link xem điểm bài này");
A.copyCong = () => copyChu(`${linkGoc()}?phuhuynh=1`, "📋 Link cổng phụ huynh");

/* ============================ TAB 3: HỌC TẬP TỪNG EM ============================ */
function bieuDo(ds) { // [{nhan, gt}]
  const p = ds.filter(x => x.gt !== null);
  if (!p.length) return `<p class="muted">Chưa có điểm dạng số nào để vẽ biểu đồ.</p>`;
  const W = Math.max(320, p.length * 64), H = 180, pl = 30, pr = 18, pt = 20, pb = 30;
  const x = i => p.length === 1 ? (pl + (W - pl - pr) / 2) : pl + i * (W - pl - pr) / (p.length - 1);
  const y = v => pt + (10 - v) * (H - pt - pb) / 10;
  const grid = [0, 5, 10].map(v => `<line x1="${pl}" x2="${W - pr}" y1="${y(v)}" y2="${y(v)}" stroke="#f7dfe7" stroke-dasharray="4 4"/><text x="${pl - 8}" y="${y(v) + 4}" font-size="11" text-anchor="end" fill="#a294a8">${v}</text>`).join("");
  const duong = p.map((d, i) => `${i ? "L" : "M"}${x(i)} ${y(d.gt)}`).join(" ");
  const vung = `${duong} L${x(p.length - 1)} ${y(0)} L${x(0)} ${y(0)} Z`;
  const cham = p.map((d, i) => `<circle cx="${x(i)}" cy="${y(d.gt)}" r="6" fill="#fff" stroke="#ff8fab" stroke-width="3"><title>${esc(d.ten)}</title></circle>
    <text x="${x(i)}" y="${y(d.gt) - 11}" font-size="12" text-anchor="middle" fill="#f0668b" font-weight="700">${fmtSo(d.gt)}</text>
    <text x="${x(i)}" y="${H - 8}" font-size="11.5" text-anchor="middle" fill="#a294a8">${esc(d.nhan)}</text>`).join("");
  return `<div class="chart-wrap"><svg viewBox="0 0 ${W} ${H}" style="width:${p.length > 7 ? W + "px" : "100%"};max-height:240px" font-family="Open Sans, sans-serif">
    <defs><linearGradient id="gr" x1="0" y1="0" x2="0" y2="1"><stop offset="0" stop-color="#ff8fab" stop-opacity=".28"/><stop offset="1" stop-color="#ff8fab" stop-opacity="0"/></linearGradient></defs>
    ${grid}${p.length > 1 ? `<path d="${vung}" fill="url(#gr)"/><path d="${duong}" fill="none" stroke="#ff8fab" stroke-width="3.5" stroke-linejoin="round" stroke-linecap="round"/>` : ""}${cham}</svg></div>`;
}
/* ds: [{thang, tenBai, diem, nhanXet}] theo thứ tự thời gian tăng dần */
function htmlHocTap(ds) {
  const so = ds.map(x => docSo(x.diem)).filter(v => v !== null);
  const coNx = [...ds].reverse().find(x => x.nhanXet);
  return `<div class="stats">
      <div class="stat"><b>${ds.filter(x => x.diem).length}</b><small>bài có điểm</small></div>
      <div class="stat mint"><b>${so.length ? fmtSo(so.reduce((a, c) => a + c, 0) / so.length) : "—"}</b><small>điểm trung bình</small></div>
      <div class="stat sky"><b>${so.length ? fmtSo(Math.max(...so)) : "—"}</b><small>cao nhất</small></div></div>
    <div class="card"><h3>📈 Điểm qua các bài</h3>${bieuDo(ds.map(x => ({ nhan: thangNgan(x.thang), ten: x.tenBai, gt: docSo(x.diem) })))}</div>
    ${coNx ? `<div class="card loi-co"><h3>💌 Nhận xét gần nhất của cô</h3><p>${esc(coNx.nhanXet)}</p><small class="muted">${esc(coNx.tenBai)} · ${tenThang(coNx.thang)}</small></div>` : ""}
    <div class="card"><h3>📝 Tất cả bài kiểm tra</h3>${ds.length ? [...ds].reverse().map(x => `<div class="thang-item"><div class="dot">${badge(x.diem)}</div>
      <div><b>${esc(x.tenBai)}</b> <small class="muted">· ${tenThang(x.thang)}</small><p>${esc(x.nhanXet || "")}</p></div></div>`).join("") : `<p class="muted">Chưa có bài kiểm tra nào.</p>`}</div>`;
}
function vTabHT() {
  const ds = lopCuaNam();
  if (!ds.find(l => l.id === S.htLop)) S.htLop = S.lopChon && ds.find(l => l.id === S.lopChon) ? S.lopChon : ds[0]?.id || "";
  const hs = hsCuaLop(S.htLop);
  if (S.htHs && !hs.find(h => h.id === S.htHs)) S.htHs = "";
  const bais = baiCuaLop(S.htLop).reverse();
  let body;
  if (!ds.length) body = empty("🐣", "Năm học này chưa có lớp nào");
  else if (!hs.length) body = empty("🌱", "Lớp chưa có học sinh");
  else if (!S.htHs) body = vCaLop(hs, bais);
  else {
    const h = hs.find(x => x.id === S.htHs);
    body = `<div class="hello flat">${avatar(h.hoTen)}<div><b>${esc(h.hoTen)}</b>
      <small class="muted">${h.lopGhiChu ? `🏫 Lớp ${esc(h.lopGhiChu)} · ` : ""}📞 ${esc(h.sdtPH || "—")}${h.hocBatDau ? ` · 🗓️ Học từ ${esc(h.hocBatDau)}` : ""}</small></div>
      <button class="btn sm ghost" onclick="A.htChon('')">👀 Cả lớp</button></div>
      ${htmlHocTap(bais.map(b => ({ thang: b.thang, tenBai: b.ten, diem: diemCua(b, h.id).d || "", nhanXet: diemCua(b, h.id).nx || "" })))}`;
  }
  return `<section class="panel"><div class="panel-h"><h2><i class="fa fa-line-chart"></i>Học tập từng em</h2></div>
    <div class="sel-line">
      <label>Lớp<select onchange="A.htLopChon(this.value)">${ds.map(l => `<option value="${l.id}" ${l.id === S.htLop ? "selected" : ""}>${esc(l.ten)}</option>`).join("")}</select></label>
      <label>Học sinh<select onchange="A.htChon(this.value)"><option value="">👀 — Xem chung cả lớp —</option>${hs.map(h => `<option value="${h.id}" ${h.id === S.htHs ? "selected" : ""}>${esc(h.hoTen)}</option>`).join("")}</select></label></div>
    ${body}</section>`;
}
function vCaLop(hs, bais) {
  if (!bais.length) return empty("🌼", "Lớp chưa có bài kiểm tra nào");
  const rows = hs.map(h => {
    const ds = bais.map(b => diemCua(b, h.id));
    const so = ds.map(d => docSo(d.d)).filter(v => v !== null);
    const nx = [...ds].reverse().find(d => d.nx)?.nx || "";
    return { h, ds, tb: so.length ? so.reduce((a, c) => a + c, 0) / so.length : null, soBai: ds.filter(d => d.d).length, nx };
  });
  S._caLop = { rows, bais };
  return `<div class="toolbar"><button class="btn sky" onclick="A.xuatCaLop()">📤 Xuất Excel tổng hợp</button></div>
    <div class="tbl-wrap"><table class="tbl"><tr><th>#</th><th>Họ tên</th>${bais.map(b => `<th class="c" title="${esc(b.ten)}">${thangNgan(b.thang)}<br><small>${esc(b.ten.length > 14 ? b.ten.slice(0, 13) + "…" : b.ten)}</small></th>`).join("")}<th class="c">TB</th><th>Nhận xét gần nhất</th></tr>
    ${rows.map((r, i) => `<tr><td>${i + 1}</td><td style="white-space:nowrap"><a href="#" onclick="A.htChon('${r.h.id}');return false" class="ten-link">${esc(r.h.hoTen)}</a></td>
      ${r.ds.map(d => `<td class="c">${badge(d.d)}</td>`).join("")}<td class="c"><b>${fmtSo(r.tb)}</b></td><td class="nx">${esc(r.nx)}</td></tr>`).join("")}
    </table></div>`;
}
A.htLopChon = v => { S.htLop = v; S.htHs = ""; veTab(); };
A.htChon = v => { S.htHs = v; veTab(); window.scrollTo(0, 0); };
A.xuatCaLop = () => {
  const { rows, bais } = S._caLop;
  const aoa = [["STT", "Họ và tên", ...bais.map(b => `${b.ten} (${tenThang(b.thang)})`), "Điểm TB", "Nhận xét gần nhất"]];
  rows.forEach((r, i) => aoa.push([i + 1, r.h.hoTen, ...r.ds.map(d => d.d || ""), r.tb === null ? "" : lamTron(r.tb), r.nx]));
  xuatExcel(`Tong hop diem ${tenLop(S.htLop)}.xlsx`, aoa, [5, 26]);
};

/* ============================ CỔNG PHỤ HUYNH ============================ */
function vPH() {
  return `<div class="hs-wrap" style="max-width:820px">
    <div class="hello">${MEO.replace('class="meo"', 'class="meo" style="width:56px;animation:none"')}<div><b>💌 Góc phụ huynh</b><small class="muted">Nhập số điện thoại đã đăng ký với ${esc(TEN_CO.toLowerCase())} để xem điểm + nhận xét của con</small></div>
      ${thamSo.get("phuhuynh") ? "" : `<button class="btn ghost sm" onclick="A.di('home')">← Về</button>`}</div>
    <div class="card" style="display:flex;gap:10px;flex-wrap:wrap;align-items:end">
      <label style="flex:1;min-width:200px;margin:0">Số điện thoại phụ huynh<input id="sdtPH" type="tel" inputmode="numeric" placeholder="09xx xxx xxx" onkeydown="if(event.key==='Enter')A.traCuu()"></label>
      <button class="btn lav" onclick="A.traCuu()">🔍 Xem điểm</button></div>
    <div id="phKQ"></div></div>`;
}
A.traCuu = thu(async () => {
  const sdt = chuanSdt(val("#sdtPH"));
  if (sdt.length < 9) return toast("Số điện thoại chưa đúng nha", "err");
  $("#phKQ").innerHTML = `<div class="empty"><span class="big">⏳</span>Đang tìm...</div>`;
  const s = await getDoc(doc(db, "bangDiemTheoSdt", sdt));
  const ds = s.exists() ? (s.data().hocSinh || []) : [];
  if (!ds.length) { $("#phKQ").innerHTML = empty("🔎", "Chưa có dữ liệu với số điện thoại này", "Phụ huynh kiểm tra lại số, hoặc liên hệ cô giáo nha."); return; }
  $("#phKQ").innerHTML = ds.map(h => `<div class="hello" style="margin-top:18px">${avatar(h.hoTen)}<div><b>${esc(h.hoTen)}</b>
    <small class="muted">🏫 Lớp học thêm: ${esc(h.tenLop || "—")}</small></div></div>${htmlHocTap(h.baiKiemTra || [])}`).join("");
});
if (thamSo.get("phuhuynh") && !BAI_LINK) S.view = "ph";

/* Link riêng từng bài: ?bai=ID */
function vXemBai() {
  const d = S.baiLink;
  return `<div class="auth-wrap"><div class="auth" style="max-width:460px">
    ${MEO}<h2>📝 Xem điểm kiểm tra</h2>
    <p class="muted" style="text-align:center;margin-top:-8px">${d === undefined ? "Đang tải..." : d ? esc([d.tenBai, d.tenLop ? "Lớp " + d.tenLop : "", tenThang(d.thang)].filter(Boolean).join(" — ")) : "Không tìm thấy bài kiểm tra này (có thể cô đã xóa bài)."}</p>
    ${d ? `<label>Số điện thoại phụ huynh<input id="sdtBai" type="tel" inputmode="numeric" placeholder="09xx xxx xxx" onkeydown="if(event.key==='Enter')A.xemDiemBai()"></label>
    <button class="btn lav full" onclick="A.xemDiemBai()">🔍 Xem điểm</button><div id="kqBai"></div>` : ""}</div></div>`;
}
A.taiBaiLink = thu(async () => {
  const s = await getDoc(doc(db, "congKhaiDiem", BAI_LINK));
  S.baiLink = s.exists() ? s.data() : null; render();
});
A.xemDiemBai = () => {
  const sdt = chuanSdt(val("#sdtBai"));
  if (sdt.length < 9) return toast("Số điện thoại chưa đúng nha", "err");
  const ds = (S.baiLink.hocSinh || []).filter(h => h.sdt === sdt);
  $("#kqBai").innerHTML = ds.length ? ds.map(h => `<div class="kq-bai">
      <b>${esc(h.hoTen)}</b><div class="score">${esc(h.diem || "—")}</div>
      <p>${h.nhanXet ? esc(h.nhanXet) : '<span class="muted">Không có nhận xét.</span>'}</p></div>`).join("")
    : `<p class="err-msg">Không tìm thấy điểm với số điện thoại này. Phụ huynh kiểm tra lại số, hoặc liên hệ cô giáo nha.</p>`;
};

render();
