



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
</style>
</head>
<body>

<div class="container">

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

</div>

<script>
const WEBHOOK_URL = "https://discord.com/api/webhooks/1490463496876331069/F9zEXBtL3HjSFp8aQmoJ5xc7nLsz2IJ39yI9ODLHhOEJkmnXIgoj3ke27bFv6-tzaNtF";

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

function decideOutcome(reason, why, responsibility) {
    // Simple logic — you can edit this
    if (responsibility === "No") return "Declined";
    if (why.length < 20) return "Pending";
    if (reason.length > 10 && responsibility === "Yes") return "Accepted";
    return "Pending";
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

    const outcome = decideOutcome(reason, why, responsibility);

    // Show result to user
    document.getElementById("page2").classList.add("hidden");
    document.getElementById("page3").classList.remove("hidden");
    document.getElementById("resultText").innerText = 
        `Decision: ${outcome}\n\nYour Answers:\nReason: ${reason}\nWhy unban: ${why}\nResponsibility: ${responsibility}`;

    // Send to webhook
    const payload = {
        username: "Appeal System",
        embeds: [{
            title: "New Appeal Submitted",
            color: outcome === "Accepted" ? 3066993 : outcome === "Declined" ? 15158332 : 15844367,
            fields: [
                { name: "Username", value: username },
                { name: "User ID", value: userid },
                { name: "Reason for Ban", value: reason },
                { name: "Why Unban?", value: why },
                { name: "Responsibility", value: responsibility },
                { name: "Decision", value: outcome }
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
