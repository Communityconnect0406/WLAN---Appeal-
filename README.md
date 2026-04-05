



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
        width: 440px;
        background: rgba(255,255,255,0.06);
        padding: 25px;
        border-radius: 12px;
        backdrop-filter: blur(10px);
        box-shadow: 0 0 25px rgba(0,0,0,0.6);
    }

    h2 {
        margin-top: 0;
        text-align: center;
    }

    input, textarea, select, button {
        width: 100%;
        padding: 10px;
        margin-top: 10px;
        border-radius: 6px;
        border: none;
        font-size: 14px;
        box-sizing: border-box;
    }

    textarea {
        min-height: 80px;
        resize: vertical;
    }

    button {
        background: #00c853;
        color: white;
        cursor: pointer;
        font-size: 16px;
        font-weight: 600;
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
        width: 50px;
        height: 50px;
        background: rgba(255,255,255,0.18);
        border-radius: 8px;
        cursor: pointer;
        transition: width 0.3s ease, height 0.3s ease;
        overflow: hidden;
        padding: 6px;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: flex-start;
        box-shadow: 0 0 10px rgba(0,0,0,0.5);
    }

    #adminBox span {
        font-size: 22px;
        line-height: 1;
    }

    #adminBox.expanded {
        width: 230px;
        height: 90px;
    }

    #adminInput {
        width: 100%;
        margin-top: 8px;
        padding: 8px;
        font-size: 15px;
        border-radius: 6px;
        border: none;
        display: none;
        box-sizing: border-box;
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
        <p id="resultText" style="white-space: pre-line;"></p>
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
const WEBHOOK_URL = "https://discord.com/api/webhooks/1490463496876331069/F9zEXBtL3HjSFp8aQmoJ5xc7nLsz2IJ39yI9ODLHhOEJkmnXIgoj3ke27bFv6-tzaNtF";

const BLACKLIST_TRIGGERS = [
    // sharing explicit content
    "explicit", "nsfw", "sexual content",
    // doxxing personal information
    "dox", "doxx", "doxxing", "personal information", "leaked info",
    // threatening someone to murder or kill them
    "murder", "kill you", "kill them", "death threat", "threaten to kill",
    // stealing assets
    "steal assets", "asset theft", "stole assets", "stolen assets"
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
        e.target.value = "";
        alert("Blacklist revoked locally. Reloading page.");
        location.reload();
    }
});

/* ------------------------------
   BLACKLIST HANDLING
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

function checkBlacklistOnLoad() {
    if (localStorage.getItem("blacklisted") === "true") {
        triggerBlacklist();
    }
}

checkBlacklistOnLoad();

function containsBlacklistReason(text) {
    const lower = text.toLowerCase();
    return BLACKLIST_TRIGGERS.some(word => lower.includes(word));
}

/* ------------------------------
   PAGE FLOW
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

/* ------------------------------
   DECISION LOGIC
------------------------------ */
function analyzeDecision(reason, why, responsibility) {
    const text = (reason + " " + why).toLowerCase();

    // Serious violations → blacklist handled before this, but keep logic clear
    if (responsibility === "No") {
        return {
            decision: "Declined",
            explanation:
                "Your appeal was declined because you did not take responsibility for your actions. Accountability is required before we can consider lifting a global ban."
        };
    }

    if (why.length < 20) {
        return {
            decision: "Pending",
            explanation:
                "Your appeal has been marked as pending due to limited detail in your explanation. A staff member will manually review your case for more context before a final decision is made."
        };
    }

    return {
        decision: "Accepted",
        explanation:
            "Your appeal was accepted because you showed responsibility and provided a clear explanation of the situation. Staff will now review your case and process your return to the community."
    };
}

/* ------------------------------
   SUBMIT APPEAL
------------------------------ */
function submitAppeal() {
    const username = document.getElementById("username").value.trim();
    const userid = document.getElementById("userid").value.trim();
    const reason = document.getElementById("reason").value.trim();
    const why = document.getElementById("why").value.trim();
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

    // NORMAL DECISION FLOW
    const result = analyzeDecision(reason, why, responsibility);

    document.getElementById("page2").classList.add("hidden");
    document.getElementById("page3").classList.remove("hidden");

    const resultText = `
Decision: ${result.decision}

Explanation:
${result.explanation}

Your Answers:
• Reason for ban: ${reason}
• Why unban: ${why}
• Responsibility: ${responsibility}
    `.trim();

    document.getElementById("resultText").innerText = resultText;

    // SEND TO WEBHOOK
    const payload = {
        username: "Appeal System",
        content: "<@&1490466157579210902>",
        embeds: [{
            title: "New Appeal Submitted",
            color: result.decision === "Accepted" ? 3066993 :
                   result.decision === "Declined" ? 15158332 : 15844367,
            fields: [
                { name: "Username", value: username || "Unknown", inline: false },
                { name: "User ID", value: userid || "Unknown", inline: false },
                { name: "Reason for Ban", value: reason || "None provided", inline: false },
                { name: "Why Unban?", value: why || "None provided", inline: false },
                { name: "Responsibility", value: responsibility, inline: false },
                { name: "Decision", value: result.decision, inline: false },
                { name: "Explanation", value: result.explanation, inline: false }
            ],
            timestamp: new Date()
        }]
    };

    fetch(WEBHOOK_URL, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(payload)
    }).catch(() => {
        // Fail silently on webhook errors to avoid breaking UX
    });
}
</script>

</body>
</html>
