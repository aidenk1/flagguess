---
layout: default
title: flag guessing
permalink: /leaderboard
---
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Leaderboard</title>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            background-color: #f0f8ff;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }
        table {
            width: 80%;
            border-collapse: collapse;
            margin-top: 20px;
        }
        table,
        th,
        td {
            border: 1px solid #ddd;
            text-align: left;
            padding: 8px;
        }
        th {
            background-color: #007bff;
            color: white;
        }
    </style>
</head>

<body>
    <h1>Leaderboard</h1>
    <table id="leaderboardTable">
        <thead>
            <tr>
                <th>User ID</th>
                <th>Score</th>
            </tr>
        </thead>
        <tbody id="leaderboardBody">
            <!-- Scores will be dynamically populated here -->
        </tbody>
    </table>
    <script>
        async function fetchLeaderboard() {
            const apiUrl = "http://127.0.0.1:8086/api/score"; // API endpoint to retrieve scores
            try {
                const response = await fetch(apiUrl);
                if (!response.ok) {
                    throw new Error('Failed to fetch leaderboard data');
                }
                const data = await response.json();
                displayLeaderboard(data);
            } catch (error) {
                console.error('Error fetching leaderboard data:', error);
            }
        }
        function displayLeaderboard(scores) {
            // Sort scores by score in descending order
            scores.sort((a, b) => b.score - a.score);
            // Create a map to store the top score for each user
            const topScoresMap = new Map();
            // Iterate through each score and update the top score for each user
            scores.forEach(score => {
                const userId = score.name;
                const userScore = score.score;
                // Check if the user's score is already in the map
                if (!topScoresMap.has(userId)) {
                    // If not, add it to the map
                    topScoresMap.set(userId, userScore);
                }
            });
            // Convert the map back to an array of objects
            const topScores = Array.from(topScoresMap, ([name, score]) => ({ name, score }));
            // Display the sorted and filtered leaderboard
            const leaderboardBody = document.getElementById('leaderboardBody');
            leaderboardBody.innerHTML = ''; // Clear previous entries
            topScores.forEach(score => {
                const row = document.createElement('tr');
                row.innerHTML = `
                    <td>${score.name}</td>
                    <td>${score.score}</td>
                `;
                leaderboardBody.appendChild(row);
            });
        }
        // Fetch leaderboard data when the page loads
        fetchLeaderboard();
    </script>
</body>

</html>