<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Automation Video Library</title>
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.4/css/all.min.css">
    <style>
        body { font-family: Arial, sans-serif; background-color: #f9f9f9; }
        header { background-color: #4CAF50; color: white; text-align: center; padding: 10px; }
        .tab-container { display: flex; justify-content: space-around; margin: 20px 0; }
        .tab-button { padding: 10px; cursor: pointer; background-color: #f2f2f2; border: 1px solid #ddd; text-align: center; flex: 1; }
        .tab-button.active { background-color: #4CAF50; color: white; }
        .tab { display: none; padding: 20px; background: white; border-radius: 5px; }
        .active { display: block; }
        .button { background: #4CAF50; color: white; padding: 10px; border: none; cursor: pointer; }
        table { width: 100%; margin-top: 20px; border-collapse: collapse; }
        th, td { padding: 10px; border: 1px solid #ddd; }
        th { background-color: #f2f2f2; }
        input, select { width: 100%; padding: 8px; margin-top: 5px; }
    </style>
</head>
<body>
    <header>
        <h1>Automation Video Library</h1>
    </header>
    <div class="tab-container">
        <div class="tab-button active" onclick="showTab('upload')">Upload Videos</div>
        <div class="tab-button" onclick="showTab('view')">View Videos</div>
    </div>
    <div id="upload" class="tab active">
        <h2>Upload Video</h2>
        <table>
            <tr><td>Buyer:</td><td><input type="text" id="buyer"></td></tr>
            <tr><td>Section:</td><td><input type="text" id="section"></td></tr>
            <tr><td>Operation:</td><td><input type="text" id="operation"></td></tr>
            <tr><td>Automation Level:</td><td><input type="text" id="automation"></td></tr>
            <tr><td>Video URL:</td><td><input type="text" id="videoUrl"></td></tr>
            <tr><td>SMV:</td><td><input type="text" id="smv"></td></tr>
        </table>
        <button class="button" onclick="submitData()">Submit</button>
    </div>
    <div id="view" class="tab">
        <h2>View Video</h2>
        <table>
            <tr><td>Select Buyer:</td><td><select id="buyerSelect" onchange="loadSections()"></select></td></tr>
            <tr><td>Select Section:</td><td><select id="sectionSelect" onchange="loadOperations()"></select></td></tr>
            <tr><td>Select Operation:</td><td><select id="operationSelect" onchange="loadAutomationLevels()"></select></td></tr>
            <tr><td>Select Automation Level:</td><td><select id="automationSelect"></select></td></tr>
        </table>
        <button class="button" onclick="loadVideo()">Load Video</button>
        <p id="smvDisplay"></p>
        <iframe id="videoFrame" style="display:none;width:100%;height:400px;"></iframe>
    </div>
    <script>
        const scriptURL = "https://script.google.com/macros/s/AKfycbwfX_eiBwI70js95QlHh2XZFCFg1s_FeVlnZHhLBtU4S6eSBhv8dtkQ2gyloY-ZHRCGQw/exec";

        function showTab(tabName) {
            $('.tab').removeClass('active');
            $('#' + tabName).addClass('active');
            $('.tab-button').removeClass('active');
            $('.tab-button').filter((_, el) => el.innerText.includes(tabName)).addClass('active');
        }

        function submitData() {
            let data = {
                buyer: $('#buyer').val(),
                section: $('#section').val(),
                operation: $('#operation').val(),
                automation: $('#automation').val(),
                videoUrl: $('#videoUrl').val(),
                smv: $('#smv').val()
            };
            $.post(scriptURL, data).done(() => alert('Data Submitted!'));
        }

        function loadBuyers() {
            $.get(scriptURL, function(data) {
                let buyers = [...new Set(data.map(row => row[0]))];
                $("#buyerSelect").html('<option>Select Buyer</option>' + buyers.map(b => `<option>${b}</option>`).join(""));
            });
        }

        function loadSections() {
            let buyer = $('#buyerSelect').val();
            $.get(scriptURL, function(data) {
                let sections = [...new Set(data.filter(row => row[0] === buyer).map(row => row[1]))];
                $("#sectionSelect").html('<option>Select Section</option>' + sections.map(s => `<option>${s}</option>`).join(""));
                $("#operationSelect").html('<option>Select Operation</option>'); // Reset operations
                $("#automationSelect").html('<option>Select Automation Level</option>'); // Reset automation levels
                $('#smvDisplay').text(''); // Reset SMV display
            });
        }

        function loadOperations() {
            let buyer = $('#buyerSelect').val();
            let section = $('#sectionSelect').val();
            $.get(scriptURL, function(data) {
                let operations = [...new Set(data.filter(row => row[0] === buyer && row[1] === section).map(row => row[2]))];
                $("#operationSelect").html('<option>Select Operation</option>' + operations.map(o => `<option>${o}</option>`).join(""));
                $("#automationSelect").html('<option>Select Automation Level</option>'); // Reset automation levels
                $('#smvDisplay').text(''); // Reset SMV display
            });
        }

        function loadAutomationLevels() {
            let operation = $('#operationSelect').val();
            $.get(scriptURL, function(data) {
                let automationLevels = [...new Set(data.filter(row => row[2] === operation).map(row => row[3]))];
                $("#automationSelect").html('<option>Select Automation Level</option>' + automationLevels.map(a => `<option>${a}</option>`).join(""));
            });
        }

        function convertYouTubeURL(url) {
            let videoId = url.match(/(?:v=|youtu\.be\/|\/embed\/|\/v\/|\/watch\?v=|\/shorts\/)([a-zA-Z0-9_-]{11})/);
            if (videoId) {
                return `https://www.youtube.com/embed/${videoId[1]}`;
            } else {
                // Handle YouTube Shorts specifically
                let shortId = url.match(/(?:youtube\.com\/shorts\/|youtu\.be\/)([a-zA-Z0-9_-]+)/);
                return shortId ? `https://www.youtube.com/embed/${shortId[1]}` : url;
            }
        }

        function convertDriveURL(url) {
            let match = url.match(/(?:drive\.google\.com\/file\/d\/|id=)([a-zA-Z0-9_-]+)/);
            return match ? `https://drive.google.com/file/d/${match[1]}/preview` : url;
        }

        function loadVideo() {
            let buyer = $('#buyerSelect').val();
            let section = $('#sectionSelect').val();
            let operation = $('#operationSelect').val();
            let automation = $('#automationSelect').val();
            $.get(scriptURL, function(data) {
                let videoEntry = data.find(row => row[0] === buyer && row[1] === section && row[2] === operation && row[3] === automation);
                if (videoEntry) {
                    let videoUrl = videoEntry[4]; // Assuming video URL is in the fourth column
                    if (videoUrl.includes("youtube.com") || videoUrl.includes("youtu.be")) {
                        $("#videoFrame").attr("src", convertYouTubeURL(videoUrl)).show();
                        $('#smvDisplay').text('SMV: ' + videoEntry[5]);
                    } else if (videoUrl.includes("drive.google.com")) {
                        $("#videoFrame").attr("src", convertDriveURL(videoUrl)).show();
                        $('#smvDisplay').text('SMV: ' + videoEntry[5]);
                    } else {
                        // Handle other video types if necessary
                        $("#videoFrame").attr("src", videoUrl).show();
                        $('#smvDisplay').text('SMV: ' + videoEntry[5]);
                    }
                } else {
                    $("#smvDisplay").text("No video found for the selected Buyer, Section, Operation & Automation Level");
                }
            });
        }

        $(document).ready(loadBuyers);
    </script>
</body>
</html>
