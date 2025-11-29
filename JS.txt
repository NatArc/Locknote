/* == Helper storage functions == */
function getAccounts() {
    return JSON.parse(localStorage.getItem("accounts")) || [];
}
function saveAccounts(accounts) {
    localStorage.setItem("accounts", JSON.stringify(accounts));
}
function setCurrentUser(email) {
    if (email) localStorage.setItem("currentUserEmail", email);
    else localStorage.removeItem("currentUserEmail");
}
function getCurrentUserEmail() {
    return localStorage.getItem("currentUserEmail");
}
function findAccountByEmail(email) {
    return getAccounts().find(a => a.email === email);
}
function updateAccount(updated) {
    const accounts = getAccounts();
    const idx = accounts.findIndex(a => a.email === updated.email);
    if (idx >= 0) {
        accounts[idx] = updated;
        saveAccounts(accounts);
    }
}

/* ================== View switching ================== */
function showView(id) {
    document.querySelectorAll(".view").forEach(v => v.classList.remove("active"));
    const v = document.getElementById(id);
    if (v) v.classList.add("active");
    renderForView(id);
}

/* ================== Render / fill content ================== */
function renderForView(id) {
    const email = getCurrentUserEmail();
    if (id === "helloView") {
        const acc = findAccountByEmail(email);
        document.getElementById("helloText").textContent = acc ? `Hello, ${acc.name || "!"}` : "Hello!";
    } else if (id === "accountView") {
        populateAccountInfo();
    }
}

/* ================== Account page fields ================== */
function populateAccountInfo() {
    const email = getCurrentUserEmail();
    const acc = findAccountByEmail(email);
    if (!acc) return;
    document.getElementById("acctName").value = acc.name || "";
    document.getElementById("acctEmail").value = acc.email;
    document.getElementById("acctPasswordMasked").value = "*".repeat(acc.password.length);
}

/* ================== FIX: Global signup step state (so reset function can access) ================== */
let currentStep = 1;            // FIX: made global
const totalSteps = 3;          // FIX: made global

/* ================== FIX: Reset functions ================== */
function resetLoginForm() {
    const loginPwd = document.getElementById("loginPassword");
    const loginErr = document.getElementById("loginError");
    if (loginPwd) loginPwd.value = "";
    if (loginErr) loginErr.textContent = "";
}

function resetSignupForm() {
    // FIX: Reset step state and UI
    currentStep = 1;
    // hide all step content and step indicators, then activate step1
    for (let i = 1; i <= totalSteps; i++) {
        const content = document.getElementById(`step${i}Content`);
        const step = document.getElementById(`step${i}`);
        if (content) content.classList.remove("active");
        if (step) step.classList.remove("active");
    }
    const step1Content = document.getElementById("step1Content");
    const step1 = document.getElementById("step1");
    if (step1Content) step1Content.classList.add("active");
    if (step1) step1.classList.add("active");

    // reset progress bar and currentStep indicator
    const progressFill = document.getElementById("progressFill");
    if (progressFill) progressFill.style.width = `${(1 / totalSteps) * 100}%`;
    const currentStepEl = document.getElementById("currentStep");
    if (currentStepEl) currentStepEl.textContent = currentStep;

    // clear signup inputs and errors
    const fields = ["name", "email", "password", "confirmPassword"];
    fields.forEach(id => {
        const el = document.getElementById(id);
        if (el) el.value = "";
    });
    const errors = ["nameError", "emailError", "passwordMatchError"];
    errors.forEach(id => {
        const el = document.getElementById(id);
        if (el) el.textContent = "";
    });

    // clear password requirements visuals
    const reqIds = ["reqLength", "reqUpperLower", "reqNumber"];
    reqIds.forEach(id => {
        const el = document.getElementById(id);
        if (el) el.classList.remove("valid");
    });

    // reset review area
    const reviewName = document.getElementById("reviewName");
    const reviewEmail = document.getElementById("reviewEmail");
    if (reviewName) reviewName.textContent = "";
    if (reviewEmail) reviewEmail.textContent = "";
}

/* ================== DOMContentLoaded ================== */
document.addEventListener("DOMContentLoaded", () => {
    // Switch views
    document.getElementById("toSignup").addEventListener("click", e => { 
        e.preventDefault(); 
        resetSignupForm();            // FIX: reset when opening signup
        showView("signupView"); 
    });
    document.getElementById("toSignin").addEventListener("click", e => { 
        e.preventDefault(); 
        showView("signinView"); 
    });

    // Top-right account buttons
    document.getElementById("accountBtn").addEventListener("click", () => showView("accountView"));
    document.getElementById("accountBtn2").addEventListener("click", () => showView("accountView"));

    // ================== FIX: LOGOUT CLEARS PASSWORD + ERROR + SIGNUP ==================
    document.getElementById("logoutBtn").addEventListener("click", () => { 
        setCurrentUser(null);
        resetLoginForm();       // FIX
        resetSignupForm();      // FIX: also reset signup (avoids leftover step state)
	document.getElementById("notesPanel").classList.remove("active");
        showView("signinView"); 
    });

    document.getElementById("logoutBtn2").addEventListener("click", () => { 
        setCurrentUser(null);
        resetLoginForm();       // FIX
        resetSignupForm();      // FIX
	document.getElementById("notesPanel").classList.remove("active");
        showView("signinView"); 
    });

    // Back to hello
    document.getElementById("backToHello").addEventListener("click", () => showView("helloView"));

    // Toggle password visibility
    document.querySelectorAll(".toggle-password").forEach(btn => {
        btn.addEventListener("click", () => {
            const target = document.getElementById(btn.dataset.target);
            if (!target) return;
            target.type = target.type === "password" ? "text" : "password";
            btn.textContent = target.type === "password" ? "👁️" : "🙈";
        });
    });

    /* ================== LOGIN ================== */
    document.getElementById("loginForm").addEventListener("submit", e => {
        e.preventDefault();

        const email = document.getElementById("loginEmail").value.trim();
        const password = document.getElementById("loginPassword").value;
        const acc = findAccountByEmail(email);

        if (acc && acc.password === password) {

            // FIX: Clear error when login succeeds
            document.getElementById("loginError").textContent = "";

            setCurrentUser(email);
            showView("helloView");
        } else {
            document.getElementById("loginError").textContent = "Incorrect email or password.";
        }
    });

    /* ================== SIGNUP Steps ================== */
    // currentStep and totalSteps are now global (above) so resetSignupForm can set them
    document.getElementById("continueStep1").addEventListener("click", () => {
        const name = document.getElementById("name").value.trim();
        const email = document.getElementById("email").value.trim();
        const accounts = getAccounts();
        let isValid = true;

        if (!name) { 
            document.getElementById("nameError").textContent = "Name is required."; 
            isValid = false; 
        } else document.getElementById("nameError").textContent = "";

        if (!email || !email.includes("@") || !email.includes(".")) { 
            document.getElementById("emailError").textContent = "Please enter a valid email."; 
            isValid = false; 
        } else if (accounts.some(acc => acc.email === email)) { 
            document.getElementById("emailError").textContent = "An account with this email already exists."; 
            isValid = false; 
        } else document.getElementById("emailError").textContent = "";

        if (isValid) nextStep();
    });

    document.getElementById("continueStep2").addEventListener("click", () => {
        const p = document.getElementById("password").value;
        const c = document.getElementById("confirmPassword").value;
        let isValid = true;

        const lengthValid = p.length >= 8;
        const upperLowerValid = /[a-z]/.test(p) && /[A-Z]/.test(p);
        const numberValid = /\d/.test(p);

        if (!lengthValid || !upperLowerValid || !numberValid) { 
            alert("Please ensure your password meets all requirements."); 
            isValid = false; 
        }

        if (p !== c) { 
            document.getElementById("passwordMatchError").textContent = "Passwords do not match."; 
            isValid = false; 
        } else document.getElementById("passwordMatchError").textContent = "";

        if (isValid) nextStep();
    });

    function nextStep() {
        // guard: ensure we don't exceed totalSteps
        if (currentStep >= totalSteps) return;

        const prev = currentStep;
        const prevContent = document.getElementById(`step${prev}Content`);
        const prevStep = document.getElementById(`step${prev}`);
        if (prevContent) prevContent.classList.remove("active");
        if (prevStep) prevStep.classList.remove("active");

        currentStep++;

        const newContent = document.getElementById(`step${currentStep}Content`);
        const newStep = document.getElementById(`step${currentStep}`);
        if (newContent) newContent.classList.add("active");
        if (newStep) newStep.classList.add("active");

        document.getElementById("currentStep").textContent = currentStep;
        document.getElementById("progressFill").style.width = `${(currentStep / totalSteps) * 100}%`;

        if (currentStep === 3) {
            document.getElementById("reviewName").textContent = document.getElementById("name").value.trim();
            document.getElementById("reviewEmail").textContent = document.getElementById("email").value.trim();
        }
    }

    document.getElementById("password").addEventListener("input", function () {
        const p = this.value;
        document.getElementById("reqLength").classList.toggle("valid", p.length >= 8);
        document.getElementById("reqUpperLower").classList.toggle("valid", /[a-z]/.test(p) && /[A-Z]/.test(p));
        document.getElementById("reqNumber").classList.toggle("valid", /\d/.test(p));
    });

    document.getElementById("signupForm").addEventListener("submit", e => {
        e.preventDefault();

        const name = document.getElementById("name").value.trim();
        const email = document.getElementById("email").value.trim();
        const password = document.getElementById("password").value;

        const accounts = getAccounts();
        accounts.push({ name, email, password });
        saveAccounts(accounts);

        setCurrentUser(email);

        // FIX: reset signup form after successful creation so next time it's fresh
        resetSignupForm();

        showView("helloView");
    });

    /* ================== ACCOUNT PAGE ================== */
    const acctNameEl = document.getElementById("acctName");
    let saveTimer = null;

    acctNameEl.addEventListener("input", () => {
        clearTimeout(saveTimer);
        saveTimer = setTimeout(() => saveAcctName(), 800);
    });

    acctNameEl.addEventListener("blur", () => saveAcctName());

    function saveAcctName() {
        const email = getCurrentUserEmail();
        const acc = findAccountByEmail(email);
        if (!acc) return;

        const newName = acctNameEl.value.trim();
        if (newName === acc.name) return;

        acc.name = newName;
        updateAccount(acc);

        const hint = document.getElementById("acctNameSaved");
        hint.style.display = "inline-block";
        setTimeout(()=> hint.style.display = "none", 1200);

        if (document.getElementById("helloText")) {
            document.getElementById("helloText").textContent = `Hello, ${acc.name || "!"}`;
        }
    }

    /* ================== Change Password ================== */
    const pwdModal = document.getElementById("pwdModal");

    document.getElementById("changePwdBtn").addEventListener("click", () => {
        document.getElementById("pwdError").textContent = "";
        document.getElementById("curPwd").value = "";
        document.getElementById("newPwd").value = "";
        document.getElementById("confirmNewPwd").value = "";
        pwdModal.style.display = "flex";
    });

    document.getElementById("pwdCancel").addEventListener("click", () => { 
        pwdModal.style.display = "none"; 
    });

    document.getElementById("pwdUpdate").addEventListener("click", () => {
        const cur = document.getElementById("curPwd").value;
        const nw = document.getElementById("newPwd").value;
        const conf = document.getElementById("confirmNewPwd").value;

        const email = getCurrentUserEmail();
        const acc = findAccountByEmail(email);
        if (!acc) return;

        if (cur !== acc.password) {
            document.getElementById("pwdError").textContent = "Current password is incorrect.";
            return;
        }

        if (nw.length < 8 || !/[a-z]/.test(nw) || !/[A-Z]/.test(nw) || !/\d/.test(nw)) {
            document.getElementById("pwdError").textContent = "New password must be 8+ chars, include upper & lower and a number.";
            return;
        }

        if (nw !== conf) {
            document.getElementById("pwdError").textContent = "New passwords do not match.";
            return;
        }

        acc.password = nw;
        updateAccount(acc);
        populateAccountInfo();
        pwdModal.style.display = "none";
        alert("Password updated.");
    });

    /* ================== Show Hello View if already logged in ================== */
    const cur = getCurrentUserEmail();
    if (cur && findAccountByEmail(cur)) {
        showView("helloView");
    } else {
        showView("signinView");
    }

    /* ================== Live DateTime ================== */
    function updateDateTime() {
        const now = new Date();
        const options = { weekday:"short", year:"numeric", month:"short", day:"numeric", hour:"2-digit", minute:"2-digit", second:"2-digit" };
        document.getElementById("helloDateTime").textContent = now.toLocaleString(undefined, options);
    }
    setInterval(updateDateTime, 1000);

    /* ================== Notes System ================== */
    const notesBtn = document.getElementById("notesBtn");
    const notesPanel = document.getElementById("notesPanel");
    const closeNotes = document.getElementById("closeNotes");
    const addNoteBtn = document.getElementById("addNoteBtn");
    const notesList = document.getElementById("notesList");
    const noteModal = document.getElementById("noteModal");
    const noteTitleInput = document.getElementById("noteTitle");
    const noteBodyInput = document.getElementById("noteBody");
    const saveNoteBtn = document.getElementById("saveNote");
    const cancelNoteBtn = document.getElementById("cancelNote");

    let editingNoteIndex = null;

    notesBtn.addEventListener("click", () => { 
        notesPanel.classList.add("active"); 
        renderNotes(); 
    });

    closeNotes.addEventListener("click", () => { 
        notesPanel.classList.remove("active"); 
    });

    addNoteBtn.addEventListener("click", () => {
        editingNoteIndex = null;
        noteTitleInput.value = "";
        noteBodyInput.value = "";
        noteModal.style.display = "flex";
    });

    cancelNoteBtn.addEventListener("click", () => { 
        noteModal.style.display = "none"; 
    });

    saveNoteBtn.addEventListener("click", () => {
        const title = noteTitleInput.value.trim();
        const body = noteBodyInput.value.trim();

        if (!title) { 
            alert("Title is required."); 
            return; 
        }

        const email = getCurrentUserEmail();
        const acc = findAccountByEmail(email);
        if (!acc) return;

        if (!acc.notes) acc.notes = [];

        if (editingNoteIndex !== null) {
            acc.notes[editingNoteIndex] = { title, body };
        } else {
            acc.notes.push({ title, body });
        }

        updateAccount(acc);
        noteModal.style.display = "none";
        renderNotes();
    });

    function renderNotes() {
        const email = getCurrentUserEmail();
        const acc = findAccountByEmail(email);

        notesList.innerHTML = "";

        if (!acc || !acc.notes || acc.notes.length === 0) return;

        acc.notes.forEach((note, idx) => {
            const div = document.createElement("div");
            div.className = "note-card";
            div.innerHTML = `<h4>${note.title}</h4><p>${note.body}</p>`;
            div.addEventListener("click", () => {
                editingNoteIndex = idx;
                noteTitleInput.value = note.title;
                noteBodyInput.value = note.body;
                noteModal.style.display = "flex";
            });
            notesList.appendChild(div);
        });
    }
});
