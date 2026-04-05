




<html lang="en">
<head>
<meta charset="UTF-8">
<title>Appeal System</title>

<style>
    body {
        margin: 0;
        padding: 0;
        background: #0f0f0f;
        color: white;
        font-family: Arial, sans-serif;
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100vh;
        transition: background 0.4s ease;
    }

    .container {
        width: 420px;
        background: rgba(255,255,255,0.06);
        padding: 25px;
        border-radius: 12px;
        backdrop-filter: blur(10px);
    }

    input, textarea, select, button {
        width: 100%;
        padding: 10px;
        margin-top: 10px;
        border-radius: 6px;
        border: none;
    }

    button {
        background: #00c853;
        color: white;
        cursor: pointer;
        font-size: 16px;
    }

    button:hover {
        background: #00e676;
    }

    .hidden {
        display: none;
    }

    /* BLACKLIST MODE */
    .blacklisted {
        background: #8b0000 !important;
    }

    /* ADMIN BOX */
    #adminBox {
        position: fixed;
        left: 10px;
        top: 50%;
        transform: translateY(-50%);
        width: 40px;
        height: 40px;
        background: rgba(255,255,255,0.15);
        border-radius: 6px;
        cursor: pointer;
        transition: width 0.3s ease;
        overflow: hidden;
        padding: 5px;
    }

    #adminBox.expanded {
        width: 200px;
    }

    #adminInput {
        width: 100%;
        margin-top: 5px;
        padding: 5px;
        border-radius: 4px;
        border: none;
        display: none;
    }

    #adminBox.expanded #adminInput {
        display: block;
    }
</style>
</head>
<body>

<!-- ADMIN BOX -->
<div id="adminBox" onclick="toggleAdminBox()">
    <span>⚙️</span>
    <input id="adminInput" placeholder="Enter admin code">
</div>

<!-- MAIN UI -->
<div class="container" id="mainContainer">

    <!-- PAGE 1 -->
    <div id="page1">
        <h2>Appeal Form</h2>
        <input id="username" placeholder="Discord Username">
        <input id="userid" placeholder="Discord ID">
        <button onclick="goToPage2()">Continue</button>
    </div>

    <!-- PAGE 2 -->
    <div id="page2" class="hidden">
        <h2>Appeal Questions</h2>
        <textarea id="reason" placeholder="Reason for your global ban"></textarea>
        <textarea id="why" placeholder="Why should you be unbanned?"></textarea>
        <select id="responsibility">
            <option value="">Do you take responsibility?</option>
            <option value="Yes">Yes</option>
            <option value="Somewhat">Somewhat</option>
            <option value="No">No</option>
        </select>
        <button onclick="submitAppeal()">Submit Appeal</button>
    </div>

    <!-- PAGE 3 -->
    <div id="page3" class="hidden">
        <h2>Your Appeal Result</h2>
        <p id="resultText"></p>
    </div>

    <!-- BLACKLIST SCREEN -->
    <div id="blacklistScreen" class="hidden">
        <h2 style="text-align:center;">
            You’re being blacklisted from our community for serious violations of our policies and Discord’s Terms of Service standards.
        </h2>
    </div>

</div>

<script>
/* ------------------------------
   CONFIG
------------------------------ */
const WEBHOOK_URL = "YOUR_WEBHOOK_HERE"; // Insert your webhook

const BLACKLIST_TRIGGERS = [
    "explicit", "nsfw", "sexual content",
    "dox", "doxxing", "personal information",
    "murder", "kill", "death threat", "threaten",
    "steal assets", "asset theft", "stole assets"
];

/* ------------------------------
   ADMIN BOX
------------------------------ */
function toggleAdminBox() {
    const box = document.getElementById("adminBox");
    box.classList.toggle("expanded");
}

document.getElementById("adminInput").addEventListener("keyup", function(e) {
    if (e.target.value === "Revoke_02") {
        localStorage.removeItem("blacklisted");
        location.reload();
    }
});

/* ------------------------------
   BLACKLIST CHECK
------------------------------ */
function triggerBlacklist() {
    localStorage.setItem("blacklisted", "true");
    document.body.classList.add("blacklisted");
    document.getElementById("mainContainer").innerHTML = `
        <h2 style="text-align:center;">
            You’re being blacklisted from our community for serious violations of our policies and Discord’s Terms of Service standards.
        </h2>
    `;
}

function checkBlacklist() {
    if (localStorage.getItem("blacklisted") === "true") {
        triggerBlacklist();
    }
}

checkBlacklist();

/* ------------------------------
   PAGE LOGIC
------------------------------ */
function goToPage2() {
    const username = document.getElementById("username").value.trim();
    const userid = document.getElementById("userid").value.trim();

    if (!username || !userid) {
        alert("Please fill out all fields.");
        return;
    }

    document.getElementById("page1").classList.add("hidden");
    document.getElementById("page2").classList.remove("hidden");
}

function containsBlacklistReason(text) {
    const lower = text.toLowerCase();
    return BLACKLIST_TRIGGERS.some(word => lower.includes(word));
}

/* ------------------------------
   SUBMIT APPEAL
------------------------------ */
function submitAppeal() {
    const username = document.getElementById("username").value;
    const userid = document.getElementById("userid").value;
    const reason = document.getElementById("reason").value;
    const why = document.getElementById("why").value;
    const responsibility = document.getElementById("responsibility").value;

    if (!reason || !why || !responsibility) {
        alert("Please answer all questions.");
        return;
    }

    // SERIOUS VIOLATION → BLACKLIST
    if (containsBlacklistReason(reason)) {
        triggerBlacklist();
        return;
    }

    // NORMAL APPEAL
    document.getElementById("page2").classList.add("hidden");
    document.getElementById("page3").classList.remove("hidden");

    document.getElementById("resultText").innerText =
        "Your appeal has been submitted. Staff will review it shortly.";

    // SEND TO WEBHOOK
    const payload = {
        username: "Appeal System",
        content: "<@&1490466157579210902>",
        embeds: [{
            title: "New Appeal Submitted",
            color: 15844367,
            fields: [
                { name: "Username", value: username },
                { name: "User ID", value: userid },
                { name: "Reason for Ban", value: reason },
                { name: "Why Unban?", value: why },
                { name: "Responsibility", value: responsibility }
            ],
            timestamp: new Date()
        }]
    };

    fetch(WEBHOOK_URL, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(payload)
    });
}
</script>

</body>
</html>
