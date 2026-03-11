{% raw %}
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>IoTDA → OBS 数据展示</title>
    <style>
        table {width: 100%; border-collapse: collapse; margin: 20px 0;}
        th, td {border: 1px solid #ddd; padding: 8px; text-align: center;}
        th {background: #f8f8f8;}
        .error {color: red; margin: 10px 0;}
    </style>
</head>
<body>
    <h1>共享单车检测数据（IoTDA+OBS）</h1>
    <div class="error" id="errorTip"></div>
    <button onclick="loadData()">刷新数据</button>
    <table id="dataTable">
        <thead>
            <tr>
                <th>图像ID</th>
                <th>检测结果</th>
                <th>置信度</th>
                <th>上报时间</th>
            </tr>
        </thead>
        <tbody></tbody>
    </table>

    <script>
        // 指向 IoTDA 实际写入的 camera_data 文件
        const OBS_URL = "https://iotda-camera-data.obs.cn-south-1.myhuaweicloud.com/camera_data";

        function loadData() {
            const errorTip = document.getElementById("errorTip");
            errorTip.textContent = "";
            const url = OBS_URL + "?t=" + new Date().getTime();
            
            fetch(url, { mode: "cors" })
            .then(res => {
                if (!res.ok) throw new Error(`OBS 访问失败：${res.status}`);
                return res.json();
            })
            .then(data => {
                const tbody = document.querySelector("#dataTable tbody");
                tbody.innerHTML = "";
                let item = {};

                // 适配 IoTDA 原始数据格式
                if (data.services && data.services[0]?.properties) {
                    const props = data.services[0].properties;
                    item = {
                        image_id: props.image_id || '-',
                        detect_result: props.detect_result || '-',
                        detect_value: props.detect_value || '-',
                        report_time: data.header?.time_stamp || new Date().toLocaleString()
                    };
                } else {
                    item = data;
                }

                const tr = document.createElement("tr");
                tr.innerHTML = `
                    <td>${item.image_id}</td>
                    <td>${item.detect_result}</td>
                    <td>${item.detect_value}</td>
                    <td>${item.report_time}</td>
                `;
                tbody.appendChild(tr);
            })
            .catch(err => {
                errorTip.textContent = "加载失败：" + err.message;
                console.log("错误详情：", err);
            });
        }

        window.onload = loadData;
    </script>
</body>
</html>
{% endraw %}