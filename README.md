


<html lang="en">
<head>
<meta charset="UTF-8">
<title>Appeal Form</title>
<style>
    body {
        font-family: Arial, sans-serif;
        background: #0f0f0f;
        color: white;
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100vh;
        transition: background 0.4s ease;
    }
    .container {
        background: rgba(255,255,255,0.05);
        padding: 25px;
        border-radius: 12px;
        width: 420px;
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
    .blacklisted {
        background: #8b0000 !important;
    }
</style>
</head>
<body>

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
        <h2 style="color:white; text-align:center;">
            You’re being blacklisted from our community for serious violations of our policies and Discord’s Terms of Service standards.
        </h2>
    </div>

</div>

<script>
const WEBHOOK_URL = "YOUR_WEBHOOK_HERE"; // Replace with your webhook

// Serious violation triggers
const BLACKLIST_KEYWORDS = [
    "explicit", "nsfw", "sexual content", "cp",
    "dox", "doxxing", "leaked info", "personal information",
    "murder", "kill you", "death threat", "threaten",
    "steal assets", "asset theft", "stole assets"
];

// Check if user is already blacklisted locally
window.onload = () => {
    if (localStorage.getItem("blacklisted") === "true") {
        triggerBlacklistScreen();
    }
};

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
    return BLACKLIST_KEYWORDS.some(word => lower.includes(word));
}

function triggerBlacklistScreen() {
    document.body.classList.add("blacklisted");
    document.getElementById("mainContainer").innerHTML = `
        <h2 style="color:white; text-align:center;">
            You’re being blacklisted from our community for serious violations of our policies and Discord’s Terms of Service standards.
        </h2>
    `;
}

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

    // BLACKLIST CHECK
    if (containsBlacklistReason(reason)) {
        localStorage.setItem("blacklisted", "true");
        triggerBlacklistScreen();
        return;
    }

    // Normal appeal flow
    document.getElementById("page2").classList.add("hidden");
    document.getElementById("page3").classList.remove("hidden");

    document.getElementById("resultText").innerText =
        "Your appeal has been submitted. Staff will review it shortly.";

    // Send to webhook
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
