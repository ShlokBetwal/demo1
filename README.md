<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Try Your Luck</title>
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    
    <style>
        * { 
            box-sizing: border-box; 
            -webkit-tap-highlight-color: transparent; /* Fixes grey box on iOS tap */
        }
        
        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            margin: 0;
            padding: 20px;
            background-color: #f0f2f5;
            /* Prevents iOS bounce scroll */
            position: fixed;
            width: 100%;
            height: 100%;
            overflow: hidden;
        }

        h1 { 
            color: #222; 
            margin: 0 0 10px 0; 
            text-transform: uppercase; 
            font-size: clamp(1.2rem, 5vw, 1.8rem);
            text-align: center;
        }

        #status {
            font-size: clamp(1rem, 4vw, 1.4rem);
            font-weight: bold;
            margin-bottom: 20px;
            color: #444;
            min-height: 2.5em;
            text-align: center;
            padding: 0 10px;
        }

        .wheel-outer-wrap {
            position: relative;
            width: 80vw;
            height: 80vw;
            max-width: 380px;
            max-height: 380px;
            margin-bottom: 40px;
            /* Fix for iOS rendering z-index issues */
            z-index: 1;
        }

        .arrow {
            position: absolute;
            top: -15px;
            left: 50%;
            transform: translateX(-50%);
            width: 0; 
            height: 0; 
            border-left: 15px solid transparent;
            border-right: 15px solid transparent;
            border-top: 30px solid #333;
            z-index: 10;
        }

        canvas {
            width: 100%;
            height: 100%;
            border-radius: 50%;
            border: 6px solid #333;
            background-color: white;
            /* iOS Performance: Force GPU acceleration */
            -webkit-transform: translateZ(0);
            transform: translateZ(0);
            transition: -webkit-transform 5s cubic-bezier(0.1, 0, 0.1, 1);
            transition: transform 5s cubic-bezier(0.1, 0, 0.1, 1);
            will-change: transform;
        }

        #spinBtn {
            padding: 16px 50px;
            font-size: 1.3rem;
            font-weight: 900;
            background-color: #333;
            color: white;
            border: none;
            border-radius: 100px;
            cursor: pointer;
            box-shadow: 0 5px 0 #000;
            /* iOS button styling reset */
            -webkit-appearance: none;
            transition: all 0.1s ease-out;
        }

        #spinBtn:active {
            transform: translateY(3px);
            box-shadow: 0 2px 0 #000;
        }

        #spinBtn:disabled {
            background-color: #999;
            box-shadow: 0 5px 0 #666;
            opacity: 0.8;
        }
    </style>
</head>
<body>

    <h1>Try your luck here</h1>
    <div id="status">Spin the wheel!</div>

    <div class="wheel-outer-wrap">
        <div class="arrow"></div>
        <canvas id="wheel" width="800" height="800"></canvas>
    </div>

    <button id="spinBtn">SPIN</button>

    <script>
        const canvas = document.getElementById('wheel');
        const ctx = canvas.getContext('2d');
        const spinBtn = document.getElementById('spinBtn');
        const statusText = document.getElementById('status');

        const segments = [
            { label: "Free Samosa Chat", color: "#FFD700" },      
            { label: "Free French Fries", color: "#2E8B57" },     
            { label: "Better Luck Next Time", color: "#D22B2B" }  
        ];

        let currentRotation = 0;

        function drawWheel() {
            const centerX = canvas.width / 2;
            const centerY = canvas.height / 2;
            const radius = canvas.width / 2 - 10; 
            const sliceAngle = (2 * Math.PI) / 3;

            ctx.clearRect(0, 0, canvas.width, canvas.height);

            segments.forEach((seg, i) => {
                const startAngle = i * sliceAngle;
                
                ctx.beginPath();
                ctx.moveTo(centerX, centerY);
                ctx.arc(centerX, centerY, radius, startAngle, startAngle + sliceAngle);
                ctx.fillStyle = seg.color;
                ctx.fill();
                ctx.strokeStyle = "#fff";
                ctx.lineWidth = 6;
                ctx.stroke();

                ctx.save();
                ctx.translate(centerX, centerY);
                ctx.rotate(startAngle + sliceAngle / 2);
                ctx.textAlign = "center";
                ctx.textBaseline = "middle";
                ctx.fillStyle = "white";
                // High-res text for mobile
                ctx.font = "bold 44px Helvetica Neue, Helvetica, Arial, sans-serif";

                const words = seg.label.split(" ");
                if(words.length > 2) {
                    ctx.fillText(words[0] + " " + words[1], radius/1.6, -30);
                    ctx.fillText(words.slice(2).join(" "), radius/1.6, 30);
                } else {
                    ctx.fillText(seg.label, radius/1.6, 0);
                }
                ctx.restore();
            });
        }

        function spin() {
            if (spinBtn.disabled) return;
            
            spinBtn.disabled = true;
            statusText.style.color = "#444";
            statusText.innerText = "Spinning...";

            // Probability: 30% Prize (15% each), 70% Better Luck
            const rand = Math.random() * 100;
            let winningIndex;
            if (rand < 15) winningIndex = 0;
            else if (rand < 30) winningIndex = 1;
            else winningIndex = 2;

            const segmentDegrees = 120;
            const targetMidPoint = (winningIndex * segmentDegrees) + 60;
            const stopAt = (360 - targetMidPoint + 270) % 360;
            
            // iOS Smoothness: Use fewer total rotations if performance lags, but 5-8 is standard
            const extraSpins = 2160; // 6 full circles
            currentRotation += (extraSpins + stopAt) - (currentRotation % 360);

            // Webkit & Standard Transform
            canvas.style.webkitTransform = `rotate(${currentRotation}deg)`;
            canvas.style.transform = `rotate(${currentRotation}deg)`;

            setTimeout(() => {
                spinBtn.disabled = false;
                const result = segments[winningIndex].label;
                const color = segments[winningIndex].color;
                statusText.innerHTML = `Result: <span style="color:${color}">${result}</span>`;
            }, 5000);
        }

        // Handle both Click and Touch for iOS responsiveness
        spinBtn.addEventListener('click', spin);
        
        // Initial Draw
        drawWheel();

        // Prevent scrolling on touch devices when touching the wheel
        document.body.addEventListener('touchmove', function(e) {
            e.preventDefault();
        }, { passive: false });
    </script>
</body>
</html>
