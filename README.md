<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Automation Video Library</title>
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.4/css/all.min.css">
    <style>
        body { font-family: Arial, sans-serif; margin: 0; padding: 0; background-color: #f9f9f9; }
        header { background-color: #4CAF50; color: white; padding: 10px; text-align: center; }
        .tab-container { display: flex; justify-content: space-around; margin: 20px 0; }
        .tab-button { flex: 1; padding: 10px; cursor: pointer; background-color: #f2f2f2; border: 1px solid #ddd; text-align: center; margin: 0 5px; }
        .tab-button.active { background-color: #4CAF50; color: white; }
        .tab { display: none; padding: 20px; border: 1px solid #ddd; border-radius: 5px; background-color: white; }
        .active { display: block; }
        .button { background-color: #4CAF50; color: white; border: none; padding: 10px 20px; cursor: pointer; border-radius: 5px; }
        .button:hover { background-color: #45a049; }
        table { width: 100%; margin: 20px 0; border-collapse: collapse; }
        th, td { padding: 10px; border: 1px solid #ddd; text-align: left; }
        th { background-color: #f2f2f2; }
        .hidden { display: none; }
        iframe, video { width: 100%; height: 400px; margin-top: 20px; }
        h2 { text-align: center; }

        /* Responsive Styles */
        @media (max-width: 768px) {
            .tab-button { font-size: 14px; padding: 8px; }
            table { font-size: 14px; }
            .button { width: 100%; }
        }

        @media (min-width: 769px) {
            .tab-button { font-size: 16px; padding: 12px; }
            table { font-size: 16px; }
            .button { width: auto; }
        }

        /* Full-width input fields */
        input[type="text"], select {
            width: 100%;
            padding: 8px;
            margin-top: 5px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
    </style>
</head>
<body>

    <header>
        <h1>Automation Video Library</h1>
    </header>

    <div class="tab-container">
        <div class="tab-button active" onclick="showTab('upload')">
            <i class="fas fa-upload icon"></i> Upload Videos
        </div>
        <div class="tab-button" onclick="showTab('view')">
            <i class="fas fa-eye icon"></i> View Videos
        </div>
    </div>

    <div id="upload" class="tab active">
        <h2>Operation Video</h2>
        <table>
            <tr>
                <th>Field</th>
                <th>Input</th>
            </tr>
            <tr>
                <td>Buyer:</td>
                <td><input type="text" id="buyer" placeholder="Enter Buyer Name"></td>
            </tr>
            <tr>
                <td>Operation:</td>
                <td><input type="text" id="operation" placeholder="Enter Operation"></td>
            </tr>
            <tr>
                <td>Automation Level:</td>
                <td><input type="text" id="automation" placeholder="Enter Automation Level"></td>
            </tr>
            <tr>
                <td>Video Type:</td>
                <td>
                    <input type="radio" name="videoType" value="youtube" checked> YouTube
                    <input type="radio" name="videoType" value="local"> Upload Video
                </td>
            </tr>
            <tr id="youtubeInputRow">
                <td>YouTube URL:</td>
                <td><input type="text" id="videoUrl" placeholder="Paste YouTube URL"></td>
            </tr>
            <tr id="fileInputRow" class="hidden">
                <td>Upload Video:</td>
                <td>
                    <input type="file" id="videoFile" accept="video/*"><br><br>
                    <label>Or Record Video:</label>
                    <input type="file" id="captureVideo" accept="video/*" capture="user"><br><br>
                </td>
            </tr>
        </table>
        <button class="button" onclick="submitData()">Submit</button>
        <p id="uploadMsg"></p>
    </div>

    <div id="view" class="tab">
        <h2>View Video</h2>
        <table>
            <tr>
                <th>Field</th>
                <th>Input</th>
            </tr>
            <tr>
                <td>Select Buyer:</td>
                <td>
                    <select id="buyerSelect" onchange="loadOperations()">
                        <option value="">Select Buyer</option>
                    </select>
                </td>
            </tr>
            <tr>
                <td>Select Operation:</td>
                <td>
                    <select id="operationSelect" onchange="loadAutomationLevels()">
                        <option value="">Select Operation</option>
                    </select>
                </td>
            </tr>
            <tr>
                <td>Select Automation Level:</td>
                <td>
                    <select id="automationSelect" onchange="displaySMV()">
                        <option value="">Select Automation Level</option>
                    </select>
                </td>
            </tr>
        </table>
        <button class="button" onclick="loadVideo()">Load Video</button>

        <h3>SMV:</h3>
        <p id="smvDisplay"></p>

        <iframe id="videoFrame" class="hidden"></iframe>
        <video id="localVideo" class="hidden" controls></video>
    </div>

    <script>
        const scriptURL = "https://script.google.com/macros/s/AKfycbyouKa4euUnzK6ZGBeu4URdO47fRuG6ximYJnhQfZvO8lDBusX6SykgfyPfgDyjxJZIbQ/exec";  // Replace with your actual Google Apps Script URL

        $(document).ready(function () {
            loadBuyers();

            $('input[name="videoType"]').change(function () {
                if ($(this).val() === "youtube") {
                    $("#youtubeInputRow").removeClass("hidden");
                    $("#fileInputRow").addClass("hidden");
                } else {
                    $("#youtubeInputRow").addClass("hidden");
                    $("#fileInputRow").removeClass("hidden");
                }
            });
        });

        function showTab(tabName) {
            $('.tab').removeClass('active');
            $('#' + tabName).addClass('active');
            $('.tab-button').removeClass('active');
            $('.tab-button').filter(function() { return $(this).text().toLowerCase().includes(tabName.toLowerCase()); }).addClass('active');
        }

        function submitData() {
            let buyer = $("#buyer").val().trim();
            let operation = $("#operation").val().trim();
            let automation = $("#automation").val().trim();
            let videoType = $('input[name="videoType"]:checked').val();

            if (!buyer || !operation || !automation) {
                $("#uploadMsg").text("All fields are required!").css("color", "red");
                return;
            }

            if (videoType === "youtube") {
                let videoUrl = $("#videoUrl").val().trim();
                if (!videoUrl) {
                    $("#uploadMsg").text("YouTube URL is required!").css("color", "red");
                    return;
                }

                $.post(scriptURL, { buyer, operation, automation, videoUrl })
                    .done(() => {
                        $("#uploadMsg").text("Data Submitted!").css("color", "green");
                        loadBuyers();
                    })
                    .fail(() => $("#uploadMsg").text("Error submitting data!").css("color", "red"));
            } else {
                let file = $("#videoFile")[0].files[0] || $("#captureVideo")[0].files[0];
                if (!file) {
                    $("#uploadMsg").text("Please select or capture a file!").css("color", "red");
                    return;
                }

                let reader = new FileReader();
                reader.readAsDataURL(file);
                reader.onloadend = function () {
                    let base64Data = reader.result.split(",")[1];

                    $.post(scriptURL, {
                        buyer,
                        operation,
                        automation,
                        fileName: file.name,
                        contents: base64Data,
                        type: file.type
                    })
                    .done(() => {
                        $("#uploadMsg").text("File Uploaded!").css("color", "green");
                        loadBuyers();
                    })
                    .fail(() => $("#uploadMsg").text("Error uploading file!").css("color", "red"));
                };
            }
        }

        function loadBuyers() {
            $.get(scriptURL, function(data) {
                let buyers = [...new Set(data.map(row => row[0]))];
                $("#buyerSelect").html('<option value="">Select Buyer</option>' + buyers.map(b => `<option>${b}</option>`).join(""));
            });
        }

        function loadOperations() {
            let selectedBuyer = $("#buyerSelect").val();
            $.get(scriptURL, function(data) {
                let operations = data.filter(row => row[0] === selectedBuyer).map(row => row[1]);
                // Use Set to ensure unique operations
                let uniqueOperations = [...new Set(operations)];
                $("#operationSelect").html('<option value="">Select Operation</option>' + uniqueOperations.map(op => `<option>${op}</option>`).join(""));
                $("#automationSelect").html('<option value="">Select Automation Level</option>'); // Reset automation levels
                $("#smvDisplay").text(""); // Reset SMV display
            });
        }

        function loadAutomationLevels() {
            let selectedBuyer = $("#buyerSelect").val();
            let selectedOperation = $("#operationSelect").val();
            $.get(scriptURL, function(data) {
                let automationLevels = data.filter(row => row[0] === selectedBuyer && row[1] === selectedOperation).map(row => row[2]); // Assuming automation level is in the third column
                $("#automationSelect").html('<option value="">Select Automation Level</option>' + [...new Set(automationLevels)].map(level => `<option>${level}</option>`).join(""));
                $("#smvDisplay").text(""); // Reset SMV display
            });
        }

        function displaySMV() {
            let selectedBuyer = $("#buyerSelect").val();
            let selectedOperation = $("#operationSelect").val();
            let selectedAutomation = $("#automationSelect").val();
            
            $.get(scriptURL, function(data) {
                let smvEntry = data.find(row => row[0] === selectedBuyer && row[1] === selectedOperation && row[2] === selectedAutomation);
                
                if (smvEntry) {
                    let smvValue = smvEntry[3]; // Assuming SMV is in the fourth column
                    $("#smvDisplay").text(smvValue);
                } else {
                    $("#smvDisplay").text("No SMV found for the selected Buyer, Operation & Automation Level");
                }
            });
        }

        function loadVideo() {
            let selectedBuyer = $("#buyerSelect").val();
            let selectedOperation = $("#operationSelect").val();
            let selectedAutomation = $("#automationSelect").val();
            
            $.get(scriptURL, function(data) {
                let videoEntry = data.find(row => row[0] === selectedBuyer && row[1] === selectedOperation && row[2] === selectedAutomation);
                
                if (videoEntry) {
                    let videoUrl = videoEntry[4]; // Assuming video URL is in the fifth column

                    if (videoUrl.includes("youtube.com") || videoUrl.includes("youtu.be")) {
                        $("#videoFrame").attr("src", convertYouTubeURL(videoUrl)).removeClass("hidden");
                        $("#localVideo").addClass("hidden");
                    } else if (videoUrl.includes("drive.google.com")) {
                        let fileId = extractDriveFileId(videoUrl);
                        if (fileId) {
                            let embeddedUrl = `https://drive.google.com/file/d/${fileId}/preview`;
                            $("#videoFrame").attr("src", embeddedUrl).removeClass("hidden");
                            $("#localVideo").addClass("hidden");
                        } else {
                            $("#uploadMsg").text("Invalid Google Drive link").css("color", "red");
                        }
                    }
                } else {
                    $("#uploadMsg").text("No video found for the selected Buyer, Operation & Automation Level").css("color", "red");
                }
            });
        }

        function extractDriveFileId(url) {
            let match = url.match(/(?:drive\.google\.com\/file\/d\/|id=)([a-zA-Z0-9_-]+)/);
            return match ? match[1] : null;
        }

        function convertYouTubeURL(url) {
            let videoId = url.match(/(?:v=|youtu\.be\/)([a-zA-Z0-9_-]{11})/);
            return videoId ? `https://www.youtube.com/embed/${videoId[1]}` : url;
        }
    </script>

</body>
</html>
