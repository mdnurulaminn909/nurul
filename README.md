<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Convert Files - www.converts.childnamefinder.com</title>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; padding: 20px; }
        .container { max-width: 500px; margin: auto; padding: 20px; border: 1px solid #ddd; border-radius: 5px; }
        .upload-box { border: 2px dashed #007bff; padding: 20px; cursor: pointer; }
        .convert-btn { background: #007bff; color: #fff; border: none; padding: 10px; cursor: pointer; margin-top: 10px; }
        .progress { display: none; margin-top: 10px; }
    </style>
</head>
<body>

    <div class="container">
        <h1>Convert Files</h1>
        <p>Select a file and choose a format to convert.</p>

        <!-- File Upload -->
        <input type="file" id="fileInput" accept=".pdf,.docx,.jpg,.png,.mp4,.mp3,.xlsx,.pptx">
        
        <!-- Format Selection -->
        <select id="formatSelect">
            <option value="pdf">PDF</option>
            <option value="docx">DOCX</option>
            <option value="jpg">JPG</option>
            <option value="png">PNG</option>
            <option value="mp4">MP4</option>
            <option value="mp3">MP3</option>
            <option value="xlsx">XLSX</option>
            <option value="pptx">PPTX</option>
        </select>
        
        <!-- Convert Button -->
        <button class="convert-btn" onclick="convertFile()">Convert</button>

        <!-- Progress -->
        <p class="progress" id="progressText">Converting...</p>

        <!-- Download Link -->
        <a id="downloadLink" style="display: none; color: blue; margin-top: 10px;"></a>
    </div>

    <script>
        async function convertFile() {
            const fileInput = document.getElementById("fileInput");
            const format = document.getElementById("formatSelect").value;
            const progressText = document.getElementById("progressText");
            const downloadLink = document.getElementById("downloadLink");

            if (fileInput.files.length === 0) {
                alert("Please select a file.");
                return;
            }

            const file = fileInput.files[0];
            const formData = new FormData();
            formData.append("file", file);
            formData.append("apikey", "YOUR_CLOUDCONVERT_API_KEY");  // Replace this with your CloudConvert API Key
            formData.append("outputformat", format);

            progressText.style.display = "block";
            progressText.innerText = "Uploading...";

            try {
                // Upload file to CloudConvert
                const response = await fetch("https://api.cloudconvert.com/v2/jobs", {
                    method: "POST",
                    headers: { "Authorization": "Bearer YOUR_CLOUDCONVERT_API_KEY", "Content-Type": "application/json" }, // Replace API Key
                    body: JSON.stringify({
                        tasks: {
                            "import-1": { operation: "import/upload" },
                            "convert-1": { operation: "convert", input: ["import-1"], output_format: format },
                            "export-1": { operation: "export/url", input: ["convert-1"] }
                        }
                    })
                });

                const job = await response.json();
                progressText.innerText = "Converting...";

                // Wait for conversion to finish
                let downloadUrl = "";
                while (!downloadUrl) {
                    await new Promise(resolve => setTimeout(resolve, 3000)); // Wait 3 seconds
                    const jobStatus = await fetch(`https://api.cloudconvert.com/v2/jobs/${job.id}`, {
                        headers: { "Authorization": "Bearer YOUR_CLOUDCONVERT_API_KEY" }
                    });
                    const jobData = await jobStatus.json();
                    const exportTask = jobData.data.tasks.find(t => t.operation === "export/url");
                    if (exportTask && exportTask.result && exportTask.result.files.length > 0) {
                        downloadUrl = exportTask.result.files[0].url;
                    }
                }

                progressText.style.display = "none";

                // Show download link
                downloadLink.href = downloadUrl;
                downloadLink.innerText = "Download Converted File";
                downloadLink.style.display = "block";

            } catch (error) {
                console.error("Error:", error);
                alert("Conversion failed. Please try again.");
                progressText.style.display = "none";
            }
        }
    </script>

</body>
</html>
