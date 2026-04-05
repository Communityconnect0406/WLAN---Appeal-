


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

// Keywords that indicate serious offenses
const SERIOUS_KEYWORDS = [
    "ddos", "dox", "doxxing", "raid", "raiding", "threat", "death",
    "ip grabber", "token grabber", "malware", "racism", "racist",
    "homophobia", "hate speech", "groom", "grooming", "sexual"
];

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

function analyzeDecision(reason, why, responsibility) {
    const text = (reason + " " + why).toLowerCase();

    let serious = SERIOUS_KEYWORDS.some(word => text.includes(word));

    if (serious) {
        return {
            decision: "Declined",
            explanation:
                "Your appeal was declined due to the severity of the actions described. The system detected keywords associated with major violations that typically result in permanent bans."
        };
    }

    if (responsibility === "No") {
        return {
            decision: "Declined",
            explanation:
                "Your appeal was declined because you did not take responsibility for your actions. Accountability is required before an appeal can be considered."
        };
    }

    if (why.length < 20) {
        return {
            decision: "Pending",
            explanation:
                "Your appeal has been marked as pending due to insufficient detail. Staff will manually review your case for further clarification."
        };
    }

    return {
        decision: "Accepted",
        explanation:
            "Your appeal was accepted because you demonstrated responsibility and provided a clear explanation. Staff will review your reintegration into the community."
    };
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

    const result = analyzeDecision(reason, why, responsibility);

    // Show result to user
    document.getElementById("page2").classList.add("hidden");
    document.getElementById("page3").classList.remove("hidden");
    document.getElementById("resultText").innerText =
        `Decision: ${result.decision}\n\nExplanation:\n${result.explanation}`;

    // Send to webhook
    const payload = {
        username: "Appeal System",
        content: "<@&1490466157579210902>", // Staff ping
        embeds: [{
            title: "New Appeal Submitted",
            color: result.decision === "Accepted" ? 3066993 :
                   result.decision === "Declined" ? 15158332 : 15844367,
            fields: [
                { name: "Username", value: username },
                { name: "User ID", value: userid },
                { name: "Reason for Ban", value: reason },
                { name: "Why Unban?", value: why },
                { name: "Responsibility", value: responsibility },
                { name: "Decision", value: result.decision },
                { name: "Explanation", value: result.explanation }
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
