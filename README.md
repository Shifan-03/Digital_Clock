# Digital_Clock
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Neon Digital Clock & Alarm</title>
    <!-- Load Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Load Tone.js for the Alarm Ringtone (Chime) -->
    <script src="https://unpkg.com/tone@14.7.58/build/Tone.js"></script>
    <style>
        /* Import a digital/tech-style font for the clock numbers */
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700&family=Orbitron:wght@400;700&display=swap');
        
        .orbitron {
            font-family: 'Orbitron', sans-serif;
            /* Default Sky Blue Neon Glow (Shadow and Color) */
            color: #38bdf8; /* sky-400 */
            text-shadow: 0 0 5px #00aaff, 0 0 15px #00aaff, 0 0 25px rgba(0, 170, 255, 0.5);
            transition: all 0.2s ease-in-out;
            /* Ensure the element can be transformed */
            will-change: transform, color, text-shadow;
        }

        .inter {
            font-family: 'Inter', sans-serif;
        }

        /* --- Custom Alarm Animations --- */
        @keyframes pulse-red {
            from { opacity: 1; }
            to { opacity: 0.7; }
        }

        @keyframes fall-down-shake {
            0% { transform: translate(0, 0) rotate(0deg); }
            25% { transform: translate(-2px, 3px) rotate(-0.5deg); }
            50% { transform: translate(2px, -3px) rotate(0.5deg); }
            75% { transform: translate(-2px, 3px) rotate(-0.5deg); }
            100% { transform: translate(0, 0) rotate(0deg); }
        }
        
        /* Alarm Active Class - Combines Glow Pulse and Physical Shake */
        .alarm-active {
            color: #f87171; /* red-400 */
            text-shadow: 0 0 5px #ef4444, 0 0 20px #ef4444, 0 0 40px rgba(239, 68, 68, 0.7);
            animation: pulse-red 0.5s infinite alternate, fall-down-shake 0.15s infinite;
        }
        /* --- End Custom Alarm Animations --- */

        /* Custom Button Styling - Highlighted Side Button Color */
        .btn-highlight {
            background-color: #0ea5e9; 
            border: 2px solid #38bdf8; 
            color: #e0f2fe; 
            text-shadow: 0 0 4px #00aaff; 
            transition: background-color 0.15s ease, transform 0.1s ease, box-shadow 0.2s ease;
            box-shadow: 0 0 10px rgba(56, 189, 248, 0.8), 0 0 20px rgba(14, 165, 236, 0.5);
        }
        .btn-highlight:hover {
            background-color: #38bdf8; 
            box-shadow: 0 0 15px rgba(56, 189, 248, 1), 0 0 30px rgba(14, 165, 236, 0.7);
            transform: translateY(-1px);
        }
        .btn-highlight:active {
            background-color: #0c4a6e; 
            box-shadow: 0 0 5px rgba(14, 165, 236, 0.4);
            transform: translateY(1px);
        }

        /* Input Styles */
        .segment-base {
            @apply flex flex-col flex-1 min-w-[70px] transition duration-200 ease-in-out; 
        }
        
        .input-field-dark {
            @apply bg-gray-800 border-2 border-gray-600 rounded-lg p-2 mt-1 shadow-inner; 
            transition: all 0.2s ease;
        }

        .input-field-dark:focus-within {
            border-color: #38bdf8; 
            box-shadow: 0 0 5px #38bdf8, inset 0 0 5px rgba(56, 189, 248, 0.3);
        }

        .input-style {
            color: #38bdf8; 
            text-shadow: 0 0 5px rgba(56, 189, 248, 0.5); 
            @apply text-xl text-center font-bold bg-transparent outline-none border-none placeholder:text-gray-500 orbitron w-full;
        }
        
        .select-style {
            color: #38bdf8; 
            text-shadow: 0 0 5px rgba(56, 189, 248, 0.5); 
            @apply text-xl text-center font-bold bg-transparent outline-none border-none appearance-none orbitron w-full h-full cursor-pointer;
        }
        
        .label-style {
            @apply text-xs text-gray-400 font-semibold mb-1; 
        }
        
        /* Prayer Times Box Styling */
        .prayer-box {
            @apply bg-gray-900 p-4 rounded-xl border-2 border-green-600 shadow-2xl shadow-green-900/50 mb-6;
        }
        .prayer-countdown-glow {
            color: #4ade80; /* green-400 */
            text-shadow: 0 0 5px #4ade80, 0 0 15px #16a34a, 0 0 20px rgba(22, 163, 74, 0.6);
        }

        html, body {
            height: 100%;
            margin: 0;
            overflow-x: hidden;
        }

    </style>
</head>
<body class="bg-gray-950 text-gray-100 inter flex flex-col items-center justify-center p-4">

    <!-- Clock Container Card -->
    <div class="bg-gray-900 p-8 md:p-16 rounded-3xl shadow-2xl w-full max-w-lg text-center border-4 border-sky-800/50">
        
        <h1 class="text-3xl sm:text-4xl font-light mb-1 text-sky-400">
            Digital Time Console
        </h1>
        <p class="text-xs mb-5 text-sky-700 font-medium inter">Tongi, Dhaka, Bangladesh</p>

        <!-- Time Display -->
        <div id="time-display" 
             class="orbitron text-5xl sm:text-6xl md:text-7xl font-bold tracking-tight leading-none">
            --:--:--
        </div>

        <!-- AM/PM indicator and Date -->
        <div class="flex flex-col items-center mt-4 mb-6">
             <div id="period-display" 
                  class="text-xl md:text-3xl font-extrabold text-sky-500 mb-2">
             </div>
             <div id="date-display" 
                  class="text-base md:text-xl font-extralight text-gray-400">
                Loading Date...
             </div>
        </div>

        <!-- PRAYER TIMES SECTION (NEW) -->
        <div class="prayer-box">
            <h2 class="text-xl font-semibold mb-3 text-green-400">
                Next Prayer
            </h2>
            <div id="prayer-info-status" class="text-xs italic mb-3 text-gray-400">
                Loading prayer times...
            </div>
            <div id="next-prayer-name" class="text-lg font-medium text-green-200 mb-2">
                --
            </div>
            <div id="next-prayer-countdown" 
                 class="orbitron text-4xl font-extrabold prayer-countdown-glow">
                --:--:--
            </div>
        </div>
        <!-- END PRAYER TIMES SECTION -->
        
        <!-- Alarm Control Panel -->
        <div class="pt-6 border-t border-sky-800/50">
            
            <div class="bg-gray-900 p-6 rounded-xl border-2 border-sky-600 shadow-2xl shadow-sky-900/50">
                <h2 class="text-xl font-semibold mb-3 text-sky-400">
                    Set Alarm (Target Time)
                </h2>
                <div id="alarm-status" class="sm:text-sm text-xs italic mb-4 text-gray-400">
                    Alarm is not set.
                </div>

                <!-- H:M AM/PM Dynamic Segments -->
                <div class="flex justify-center space-x-3 flex-wrap sm:flex-nowrap mb-4 items-end">
                    
                    <!-- Hour Input Segment -->
                    <div class="segment-base">
                        <label for="target-hour" class="label-style">HOUR</label>
                        <div class="input-field-dark">
                            <input type="number" id="target-hour" min="1" max="12" value="8" 
                                placeholder="H" 
                                class="input-style">
                        </div>
                    </div>

                    <!-- Minute Input Segment -->
                    <div class="segment-base">
                        <label for="target-minute" class="label-style">MINUTE</label>
                        <div class="input-field-dark">
                            <input type="number" id="target-minute" min="0" max="59" value="30" 
                                placeholder="M" 
                                class="input-style">
                        </div>
                    </div>

                    <!-- AM/PM Select Segment -->
                    <div class="segment-base">
                        <label for="target-ampm" class="label-style">PERIOD</label>
                        <div class="input-field-dark">
                            <select id="target-ampm" class="select-style">
                                <option value="AM">AM</option>
                                <option value="PM" selected>PM</option>
                            </select>
                        </div>
                    </div>
                </div>
            </div> <!-- End ALARM BOX -->

            <div class="flex justify-center space-x-3 mt-4">
                <!-- Set Alarm Button -->
                <button onclick="setAlarm()" 
                        class="btn-highlight text-lg tracking-widest font-extrabold py-3 px-8 rounded-lg uppercase">
                    Set Alarm
                </button>
                <!-- Clear Button -->
                <button onclick="clearAlarm()" 
                        class="bg-red-600 hover:bg-red-700 active:bg-red-800 text-white font-bold py-3 px-4 rounded-lg text-sm transition duration-150 ease-in-out">
                    Stop / Clear
                </button>
            </div>
        </div>

        <!-- Attribution (NEW) -->
        <div class="mt-8 pt-4 border-t border-sky-800/50">
            <p class="text-sm font-light text-gray-500 orbitron">
                Created by <span class="text-sky-400 font-medium">Shifan</span>
            </p>
        </div>
        <!-- END Attribution -->
    </div>

    <!-- JavaScript Logic -->
    <script>
        // Global variables for alarm state
        let ALARM_IS_SET = false;
        let ALARM_HOUR = null; 
        let ALARM_MINUTE = null;
        let ALARM_TRIGGERED = false; 
        let flashTimeout = null;
        let chimeInterval = null;
        const ALARM_DURATION_MS = 15000; // 15 seconds

        // Global variables for prayer times
        let PRAYER_TIMES = null; 
        // Prayer names, excluding Sunrise as it's not a formal prayer (Salah) time
        const PRAYER_NAMES_ORDER = ['Fajr', 'Sunrise', 'Dhuhr', 'Asr', 'Maghrib', 'Isha']; 
        const LATITUDE = 23.9011; // Tongi, Dhaka, Bangladesh
        const LONGITUDE = 90.4070;
        const CALCULATION_METHOD = 1; // University of Islamic Sciences, Karachi
        const API_KEY = ""; // Required for fetch but left empty for Canvas environment

        // Get references to display elements
        const timeDisplay = document.getElementById('time-display');
        const periodDisplay = document.getElementById('period-display');
        const alarmStatus = document.getElementById('alarm-status');
        const targetHourInput = document.getElementById('target-hour');
        const targetMinuteInput = document.getElementById('target-minute');
        const targetAmpmSelect = document.getElementById('target-ampm');
        const prayerStatusEl = document.getElementById('prayer-info-status');
        const nextPrayerNameEl = document.getElementById('next-prayer-name');
        const nextPrayerCountdownEl = document.getElementById('next-prayer-countdown');
        
        // --- Tone.js Setup for Ringtone ---
        let synth;
        try {
            synth = new Tone.Synth({
                oscillator: { type: "square" },
                envelope: { attack: 0.05, decay: 0.2, sustain: 0.1, release: 0.5 }
            }).toDestination();
            
            document.body.addEventListener('click', () => {
                if (Tone.context.state !== 'running') {
                    Tone.context.resume();
                }
            }, { once: true });

        } catch (e) {
            console.error("Tone.js could not initialize:", e);
        }

        function playAlarmSound() {
            if (!synth) return;
            const chimeSequence = () => {
                const now = Tone.now();
                synth.triggerAttackRelease("C5", "8n", now);
                synth.triggerAttackRelease("E5", "8n", now + 0.25); 
                synth.triggerAttackRelease("G5", "8n", now + 0.5);
            };
            chimeSequence();
            chimeInterval = setInterval(chimeSequence, 1000);
        }
        
        function stopAlarmSound() {
            if (chimeInterval) {
                clearInterval(chimeInterval);
                chimeInterval = null;
            }
        }
        // --- End Tone.js Setup ---


        /**
         * Pads a number with a leading zero if it's a single digit.
         */
        const pad = (num) => String(num).padStart(2, '0');
        
        /**
         * Clears the alarm flag and resets display elements to normal.
         */
        function clearAlarmVisuals() {
             timeDisplay.classList.remove('alarm-active');
             alarmStatus.classList.remove('alarm-active');
             if (flashTimeout) clearTimeout(flashTimeout); 
        }

        /**
         * Determines if the set alarm time is for today or tomorrow.
         */
        function getTimeContext(alarmHour, alarmMinute) {
            const now = new Date();
            const currentTotalMinutes = now.getHours() * 60 + now.getMinutes();
            const alarmTotalMinutes = alarmHour * 60 + alarmMinute;
            
            if (alarmTotalMinutes <= currentTotalMinutes) {
                 return "Tomorrow";
            }
            return "Today";
        }

        /**
         * Sets the alarm to go off at a specific target time.
         */
        function setAlarm() {
            stopAlarmSound();
            clearAlarmVisuals();
            ALARM_TRIGGERED = false;

            const targetHour12 = parseInt(targetHourInput.value, 10);
            const targetMinute = parseInt(targetMinuteInput.value, 10);
            const ampm = targetAmpmSelect.value;

            if (isNaN(targetHour12) || isNaN(targetMinute) || targetHour12 < 1 || targetHour12 > 12 || targetMinute < 0 || targetMinute > 59) {
                alarmStatus.textContent = "Invalid time. Hours must be 1-12, Minutes 0-59.";
                alarmStatus.classList.remove('text-gray-400');
                alarmStatus.classList.add('text-red-500');
                return;
            }

            let targetHour24 = targetHour12;
            if (ampm === 'PM' && targetHour12 !== 12) {
                targetHour24 += 12;
            } else if (ampm === 'AM' && targetHour12 === 12) {
                targetHour24 = 0; 
            }

            ALARM_HOUR = targetHour24;
            ALARM_MINUTE = targetMinute;
            ALARM_IS_SET = true;
            
            const timeContext = getTimeContext(ALARM_HOUR, ALARM_MINUTE);
            
            alarmStatus.classList.remove('text-red-500', 'alarm-active');
            alarmStatus.classList.add('text-gray-400'); 
            alarmStatus.textContent = `Alarm set for ${timeContext}: ${pad(targetHour12)}:${pad(ALARM_MINUTE)} ${ampm}`;
        }
        
        /**
         * Clears the current alarm completely and stops any active ringing.
         */
        function clearAlarm() {
            ALARM_IS_SET = false;
            ALARM_HOUR = null;
            ALARM_MINUTE = null;
            ALARM_TRIGGERED = false; 

            stopAlarmSound();
            clearAlarmVisuals();
            
            alarmStatus.classList.remove('text-red-500');
            alarmStatus.classList.add('text-gray-400'); 
            alarmStatus.textContent = "Alarm is not set.";
        }
        
        // --- Prayer Time Logic ---

        let prayerFetchDate = null;

        /**
         * Fetches prayer times from AlAdhan API (only once per day).
         */
        async function fetchPrayerTimes() {
            const today = new Date().toDateString();

            // Check if already fetched for today
            if (prayerFetchDate === today && PRAYER_TIMES) {
                return;
            }

            prayerFetchDate = today;
            prayerStatusEl.textContent = "Loading prayer times...";
            nextPrayerNameEl.textContent = "--";
            nextPrayerCountdownEl.textContent = "--:--:--";

            // Get date components for the API URL
            const now = new Date();
            const day = now.getDate();
            const month = now.getMonth() + 1; // getMonth() returns 0-11
            const year = now.getFullYear();

            // AlAdhan API URL using coordinates and calculation method for Tongi/Bangladesh
            const apiUrl = `https://api.aladhan.com/v1/timings/${day}-${month}-${year}?latitude=${LATITUDE}&longitude=${LONGITUDE}&method=${CALCULATION_METHOD}&school=1&tune=0,0,0,0,0,0,0,0,0`;

            try {
                const response = await fetch(apiUrl);
                if (!response.ok) throw new Error("API call failed.");
                
                const data = await response.json();
                const timings = data.data.timings;

                // Store only necessary 24-hour time strings
                PRAYER_TIMES = {
                    date: today,
                    Fajr: timings.Fajr,
                    Sunrise: timings.Sunrise,
                    Dhuhr: timings.Dhuhr,
                    Asr: timings.Asr,
                    Maghrib: timings.Maghrib,
                    Isha: timings.Isha,
                };

                prayerStatusEl.textContent = `Tongi, Dhaka (+06:00) - Calculated`;

            } catch (error) {
                console.error("Prayer Time API Error:", error);
                prayerStatusEl.textContent = "Error loading prayer times (API rate limit or network issue).";
                PRAYER_TIMES = null; // Reset on failure
            }
        }

        /**
         * Finds the next prayer time and updates the countdown display.
         */
        function calculateNextPrayer(now) {
            if (!PRAYER_TIMES) return;

            const nowTimestamp = now.getTime();
            let nextPrayerName = 'Fajr (Tomorrow)';
            let nextPrayerTimeStr = PRAYER_TIMES.Fajr;
            let nextPrayerTimestamp = 0;
            let foundNextPrayer = false;

            // 1. Check today's prayers
            for (const name of PRAYER_NAMES_ORDER) {
                const timeStr = PRAYER_TIMES[name];
                
                // Create a Date object for the current day at the prayer time
                const [hour, minute] = timeStr.split(':').map(Number);
                const prayerDate = new Date(now);
                prayerDate.setHours(hour, minute, 0, 0); 
                const prayerTimestamp = prayerDate.getTime();

                if (prayerTimestamp > nowTimestamp) {
                    nextPrayerName = name;
                    nextPrayerTimeStr = timeStr;
                    nextPrayerTimestamp = prayerTimestamp;
                    foundNextPrayer = true;
                    break; 
                }
            }
            
            // 2. If no prayer was found, the next is Fajr tomorrow
            if (!foundNextPrayer) {
                // Set date to tomorrow
                const tomorrow = new Date(now);
                tomorrow.setDate(tomorrow.getDate() + 1); 
                
                // Set time to Fajr
                const [hour, minute] = PRAYER_TIMES.Fajr.split(':').map(Number);
                tomorrow.setHours(hour, minute, 0, 0); 
                nextPrayerTimestamp = tomorrow.getTime();
                nextPrayerTimeStr = PRAYER_TIMES.Fajr;
            }

            // 3. Calculate Countdown
            const diffMs = nextPrayerTimestamp - nowTimestamp;
            
            if (diffMs < 0) return; // Wait for the next update cycle or fetch

            const totalSeconds = Math.floor(diffMs / 1000);
            const hoursLeft = Math.floor(totalSeconds / 3600);
            const minutesLeft = Math.floor((totalSeconds % 3600) / 60);
            const secondsLeft = totalSeconds % 60;

            const countdownText = `${pad(hoursLeft)}:${pad(minutesLeft)}:${pad(secondsLeft)}`;

            // 4. Update HTML
            nextPrayerNameEl.textContent = `${nextPrayerName} at ${nextPrayerTimeStr}`;
            nextPrayerCountdownEl.textContent = countdownText;
        }


        /**
         * Main function that updates the digital clock, checks the alarm, and updates prayer countdown.
         */
        function updateClock() {
            const now = new Date();
            
            let hours = now.getHours();
            const minutes = now.getMinutes();
            const seconds = now.getSeconds();
            
            // --- PRAYER TIME UPDATE ---
            // Ensure we fetch if the day has changed
            if (!prayerFetchDate || prayerFetchDate !== now.toDateString()) {
                fetchPrayerTimes();
            }
            calculateNextPrayer(now);

            // --- TIME DISPLAY LOGIC (12-Hour Format) ---
            const period = hours >= 12 ? 'PM' : 'AM';
            let displayHours = hours % 12;
            displayHours = displayHours ? displayHours : 12; 

            const timePart = `${pad(displayHours)}:${pad(minutes)}:${pad(seconds)}`;
            timeDisplay.textContent = timePart;
            periodDisplay.textContent = period;

            const dateOptions = { 
                weekday: 'long', 
                year: 'numeric', 
                month: 'long', 
                day: 'numeric' 
            };
            const dateString = now.toLocaleDateString('en-US', dateOptions);
            document.getElementById('date-display').textContent = dateString;

            // --- ALARM CHECK LOGIC ---
            const isTriggerTime = (ALARM_HOUR === hours && ALARM_MINUTE === minutes);

            if (ALARM_IS_SET) {
                if (isTriggerTime && seconds === 0 && !ALARM_TRIGGERED) {
                    ALARM_TRIGGERED = true; 
                    
                    playAlarmSound();
                    
                    timeDisplay.classList.add('alarm-active');
                    alarmStatus.classList.add('alarm-active', 'text-red-500');
                    alarmStatus.textContent = `!!! ALARM TRIGGERED (15s) !!! Tap 'Stop / Clear' to silence.`;
                    
                    flashTimeout = setTimeout(() => {
                        stopAlarmSound();
                        clearAlarmVisuals();
                        
                        const timeContext = getTimeContext(ALARM_HOUR, ALARM_MINUTE);
                        let displayHours = ALARM_HOUR % 12;
                        displayHours = displayHours ? displayHours : 12;
                        const displayPeriod = ALARM_HOUR >= 12 ? 'PM' : 'AM';
                        
                        alarmStatus.classList.remove('text-red-500');
                        alarmStatus.classList.add('text-gray-400'); 
                        alarmStatus.textContent = `Alarm set for ${timeContext}: ${pad(displayHours)}:${pad(ALARM_MINUTE)} ${displayPeriod}`;

                    }, ALARM_DURATION_MS); 
                } 
                
                if (!isTriggerTime && ALARM_TRIGGERED) {
                    ALARM_TRIGGERED = false;
                }
            } 
            
            if (!ALARM_IS_SET) {
                stopAlarmSound();
                clearAlarmVisuals();
            }
        }

        // Initialize prayer times fetching and start the clock interval
        fetchPrayerTimes(); 
        setInterval(updateClock, 1000); 
    </script>
</body>
</html>
