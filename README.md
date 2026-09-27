<!DOCTYPE html>
<html dir="rtl" lang="he" class="dark">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>StreakMaster - מעקב רצפים והתראות יומיות</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css2?family=Rubik:wght@300;400;500;600;700;800;900&display=swap" rel="stylesheet">
    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    colors: {
                        brand: {
                            50: '#fff7ed',
                            100: '#ffedd5',
                            500: '#f97316',
                            600: '#ea580c',
                            700: '#c2410c',
                        },
                        streak: {
                            fire: '#ff4500',
                            gold: '#ffd700',
                            dark: '#0f172a',
                            card: '#1e293b'
                        }
                    },
                    fontFamily: {
                        sans: ['Rubik', 'sans-serif'],
                    },
                    animation: {
                        'pulse-slow': 'pulse 3s cubic-bezier(0.4, 0, 0.6, 1) infinite',
                        'bounce-short': 'bounce 0.6s ease-in-out 2',
                        'glow': 'glow 2s ease-in-out infinite alternate',
                    },
                    keyframes: {
                        glow: {
                            '0%': { boxShadow: '0 0 15px rgba(249, 115, 22, 0.4)' },
                            '100%': { boxShadow: '0 0 30px rgba(249, 115, 22, 0.8)' }
                        }
                    }
                }
            }
        }
    </script>
    <style>
        body {
            font-family: 'Rubik', sans-serif;
            -webkit-tap-highlight-color: transparent;
            user-select: none;
        }
        .fire-gradient {
            background: linear-gradient(135deg, #ff8c00 0%, #ff4500 50%, #dc2626 100%);
        }
        .card-glow {
            box-shadow: 0 10px 30px -10px rgba(249, 115, 22, 0.3);
        }
        .custom-scrollbar::-webkit-scrollbar {
            width: 6px;
        }
        .custom-scrollbar::-webkit-scrollbar-track {
            background: #0f172a;
        }
        .custom-scrollbar::-webkit-scrollbar-thumb {
            background: #334155;
            border-radius: 4px;
        }
    </style>
</head>
<body class="bg-slate-950 text-slate-100 min-h-screen pb-24 flex flex-col justify-between selection:bg-orange-500 selection:text-white custom-scrollbar">

    <!-- Canvas for Confetti -->
    <canvas id="confettiCanvas" class="fixed inset-0 pointer-events-none z-50 w-full h-full"></canvas>

    <!-- Toast Notification Container -->
    <div id="toastContainer" class="fixed top-16 left-1/2 -translate-x-1/2 z-50 w-11/12 max-w-sm flex flex-col gap-2 pointer-events-none"></div>

    <!-- Top Header -->
    <header class="sticky top-0 z-40 bg-slate-900/90 backdrop-blur-md border-b border-slate-800 px-4 py-3 shadow-lg">
        <div class="max-w-md mx-auto flex items-center justify-between">
            <div class="flex items-center space-x-3 space-x-reverse">
                <div class="w-10 h-10 rounded-2xl fire-gradient flex items-center justify-center text-white shadow-md shadow-orange-500/30">
                    <i class="fa-solid font-bold fa-fire text-xl animate-pulse"></i>
                </div>
                <div>
                    <h1 class="text-lg font-black tracking-wide text-white flex items-center gap-2">
                        StreakMaster
                        <span class="text-[10px] bg-orange-500/20 text-orange-400 border border-orange-500/30 px-2 py-0.5 rounded-full">פוש v1.0</span>
                    </h1>
                    <p class="text-xs text-slate-400">מעקב רצף והתראות יומיות</p>
                </div>
            </div>

            <div class="flex items-center gap-2">
                <button id="notifStatusBtn" onclick="openNotificationModal()" class="flex items-center gap-1.5 px-3 py-1.5 rounded-xl text-xs font-semibold bg-slate-800 hover:bg-slate-700 text-slate-300 border border-slate-700 transition-all">
                    <i id="notifStatusIcon" class="fa-solid fa-bell text-orange-400"></i>
                    <span id="notifStatusText">התראות</span>
                </button>
            </div>
        </div>
    </header>

    <!-- Main Container -->
    <main class="max-w-md mx-auto w-full px-4 pt-4 flex-1 flex flex-col gap-5">

        <!-- User Level & Total Streak Stats Card -->
        <div class="bg-gradient-to-br from-slate-900 via-slate-800 to-slate-900 border border-slate-800 rounded-3xl p-5 shadow-xl relative overflow-hidden card-glow">
            <div class="absolute -left-10 -bottom-10 w-32 h-32 bg-orange-500/10 rounded-full blur-2xl pointer-events-none"></div>

            <div class="flex items-center justify-between mb-4">
                <div class="flex items-center gap-3">
                    <div class="relative">
                        <div class="w-14 h-14 rounded-2xl fire-gradient flex items-center justify-center text-white font-black text-2xl shadow-lg shadow-orange-600/40">
                            <span id="levelDisplay">1</span>
                        </div>
                        <span class="absolute -bottom-1 -right-1 bg-slate-900 border border-slate-700 text-[10px] px-1.5 py-0.5 rounded-md font-bold text-orange-400">רמה</span>
                    </div>
                    <div>
                        <h2 id="rankTitle" class="font-bold text-base text-slate-100">טירון הלהבה 🔥</h2>
                        <p id="xpText" class="text-xs text-slate-400">0 / 100 XP לרמה הבאה</p>
                    </div>
                </div>

                <div class="text-center bg-slate-950/60 border border-slate-800/80 px-4 py-2 rounded-2xl">
                    <div class="flex items-center justify-center gap-1 text-orange-500">
                        <i class="fa-solid fa-fire text-lg animate-bounce-short"></i>
                        <span id="totalStreakCount" class="text-2xl font-black text-white">0</span>
                    </div>
                    <div class="text-[10px] font-medium text-slate-400">ימי רצף שיא</div>
                </div>
            </div>

            <!-- XP Progress Bar -->
            <div class="w-full bg-slate-950 rounded-full h-3 p-0.5 border border-slate-800">
                <div id="xpProgressBar" class="fire-gradient h-full rounded-full transition-all duration-500" style="width: 0%"></div>
            </div>
        </div>

        <!-- Notification Quick Banner -->
        <div id="notifBanner" class="bg-orange-500/10 border border-orange-500/30 rounded-2xl p-3 flex items-center justify-between">
            <div class="flex items-center gap-3">
                <div class="w-9 h-9 rounded-xl bg-orange-500/20 flex items-center justify-center text-orange-400">
                    <i class="fa-solid fa-bell-ring text-sm animate-pulse"></i>
                </div>
                <div>
                    <h3 class="text-xs font-bold text-orange-200">הפעל התראות פוש בנייד</h3>
                    <p class="text-[11px] text-orange-300/80">כדי שלא תשכח לבצע את המשימה היומית שלך</p>
                </div>
            </div>
            <button onclick="requestNotificationPermission()" class="px-3 py-1.5 bg-orange-500 hover:bg-orange-600 active:scale-95 text-white font-bold text-xs rounded-xl shadow-md transition-all">
                אישור
            </button>
        </div>

        <!-- Section Title & Add Button -->
        <div class="flex items-center justify-between pt-1">
            <h2 class="text-base font-bold text-slate-200 flex items-center gap-2">
                <i class="fa-solid fa-list-check text-orange-500"></i>
                הרצפים והמשימות שלי
            </h2>
            <button onclick="openAddHabitModal()" class="flex items-center gap-1.5 bg-orange-500 hover:bg-orange-600 active:scale-95 text-white text-xs font-bold px-3 py-2 rounded-xl shadow-lg shadow-orange-500/20 transition-all">
                <i class="fa-solid fa-plus"></i>
                משימה חדשה
            </button>
        </div>

        <!-- Habits List -->
        <div id="habitsList" class="flex flex-col gap-3">
            <!-- Habit Items will be dynamically inserted here -->
        </div>

        <!-- Empty State -->
        <div id="emptyState" class="hidden text-center py-10 px-4 bg-slate-900/40 border border-dashed border-slate-800 rounded-3xl">
            <div class="w-16 h-16 rounded-3xl bg-slate-800/80 flex items-center justify-center mx-auto mb-3 text-slate-500 text-2xl">
                <i class="fa-solid fa-calendar-plus"></i>
            </div>
            <h3 class="font-bold text-slate-300 mb-1">עדיין אין לך רצפים מוגדרים</h3>
            <p class="text-xs text-slate-500 mb-4">הוסף משימה ראשונה, הגדר שעת התראה ותתחיל לבנות את רצף הימים שלך!</p>
            <button onclick="openAddHabitModal()" class="bg-orange-500 hover:bg-orange-600 text-white font-bold text-xs px-4 py-2.5 rounded-xl transition-all">
                צור רצף חדש כעת
            </button>
        </div>

        <!-- Achievements & Badges Grid -->
        <div class="mt-2">
            <h2 class="text-sm font-bold text-slate-300 mb-3 flex items-center gap-2">
                <i class="fa-solid fa-trophy text-amber-400"></i>
                הישגים ותגים
            </h2>
            <div id="badgesGrid" class="grid grid-cols-4 gap-2">
                <!-- Badges will be generated via JS -->
            </div>
        </div>

    </main>

    <!-- Bottom Navigation Bar -->
    <nav class="fixed bottom-0 left-0 right-0 z-40 bg-slate-900/95 backdrop-blur-md border-t border-slate-800 py-2 px-6">
        <div class="max-w-md mx-auto flex items-center justify-around">
            <button onclick="switchTab('today')" id="tabToday" class="flex flex-col items-center gap-1 text-orange-500 font-bold text-xs">
                <i class="fa-solid fa-fire text-lg"></i>
                <span>היום</span>
            </button>
            <button onclick="triggerTestNotification()" class="flex flex-col items-center gap-1 text-slate-400 hover:text-slate-200 text-xs font-semibold">
                <i class="fa-solid fa-[#ff4500] fa-paper-plane text-lg text-orange-400 animate-bounce"></i>
                <span>בחינת פוש</span>
            </button>
            <button onclick="openNotificationModal()" class="flex flex-col items-center gap-1 text-slate-400 hover:text-slate-200 text-xs font-semibold">
                <i class="fa-solid fa-sliders text-lg"></i>
                <span>הגדרות</span>
            </button>
        </div>
    </nav>

    <!-- Modal: Add / Edit Habit -->
    <div id="habitModal" class="fixed inset-0 z-50 bg-slate-950/80 backdrop-blur-sm flex items-end sm:items-center justify-center p-0 sm:p-4 opacity-0 pointer-events-none transition-all duration-300">
        <div class="bg-slate-900 border border-slate-800 w-full max-w-md rounded-t-3xl sm:rounded-3xl p-5 shadow-2xl transform translate-y-full sm:translate-y-0 transition-transform duration-300">
            <div class="flex items-center justify-between mb-4 border-b border-slate-800 pb-3">
                <h3 id="modalTitle" class="font-black text-lg text-white flex items-center gap-2">
                    <i class="fa-solid fa-plus-circle text-orange-500"></i>
                    יצירת רצף ומשימה חדשה
                </h3>
                <button onclick="closeHabitModal()" class="w-8 h-8 rounded-full bg-slate-800 text-slate-400 hover:text-white flex items-center justify-center">
                    <i class="fa-solid fa-xmark"></i>
                </button>
            </div>

            <form id="habitForm" onsubmit="saveHabit(event)" class="flex flex-col gap-4">
                <input type="hidden" id="habitId">
                
                <div>
                    <label class="block text-xs font-bold text-slate-300 mb-1.5">שם המשימה / ההרגל</label>
                    <input type="text" id="habitTitle" required placeholder="לדוגמה: אימון כושר יומי / לימוד / שתיית מים" class="w-full bg-slate-950 border border-slate-800 rounded-xl px-3.5 py-2.5 text-sm text-white focus:outline-none focus:border-orange-500 transition-colors">
                </div>

                <div class="grid grid-cols-2 gap-3">
                    <div>
                        <label class="block text-xs font-bold text-slate-300 mb-1.5">שעת התראת פוש</label>
                        <input type="time" id="habitTime" value="20:00" required class="w-full bg-slate-950 border border-slate-800 rounded-xl px-3 py-2.5 text-sm text-white focus:outline-none focus:border-orange-500 text-center">
                    </div>
                    <div>
                        <label class="block text-xs font-bold text-slate-300 mb-1.5">אייקון ייצוגי</label>
                        <select id="habitIcon" class="w-full bg-slate-950 border border-slate-800 rounded-xl px-3 py-2.5 text-sm text-white focus:outline-none focus:border-orange-500">
                            <option value="fa-fire">🔥 אש / ספורט</option>
                            <option value="fa-book-open">📖 לימוד / קריאה</option>
                            <option value="fa-dumbbell">🏋️ אימון / כושר</option>
                            <option value="fa-glass-water">💧 שתיית מים</option>
                            <option value="fa-brain">🧠 מדיטציה / ריכוז</option>
                            <option value="fa-pen-nib">✍️ כתיבה / יומן</option>
                            <option value="fa-capsules">💊 ויטמינים / תרופה</option>
                            <option value="fa-laptop-code">💻 תכנון / עבודה</option>
                        </select>
                    </div>
                </div>

                <div class="flex items-center justify-between bg-slate-950/60 p-3 rounded-xl border border-slate-800">
                    <div class="flex items-center gap-2">
                        <i class="fa-solid fa-bell text-orange-400 text-sm"></i>
                        <span class="text-xs font-semibold text-slate-300">הפעל התראה יומית בשעה זו</span>
                    </div>
                    <label class="relative inline-flex items-center cursor-pointer">
                        <input type="checkbox" id="habitNotifToggle" checked class="sr-only peer">
                        <div class="w-11 h-6 bg-slate-800 peer-focus:outline-none rounded-full peer peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:right-[2px] after:bg-white after:border-gray-300 after:border after:rounded-full after:h-5 after:w-5 after:transition-all peer-checked:bg-orange-500"></div>
                    </label>
                </div>

                <div class="flex gap-2 pt-2">
                    <button type="button" onclick="closeHabitModal()" class="flex-1 bg-slate-800 hover:bg-slate-700 text-slate-300 font-bold text-xs py-3 rounded-xl transition-all">
                        ביטול
                    </button>
                    <button type="submit" class="flex-1 bg-orange-500 hover:bg-orange-600 text-white font-bold text-xs py-3 rounded-xl shadow-lg shadow-orange-500/25 transition-all">
                        שמור משימה
                    </button>
                </div>
            </form>
        </div>
    </div>

    <!-- Modal: Notification Settings & Guide -->
    <div id="notifModal" class="fixed inset-0 z-50 bg-slate-950/80 backdrop-blur-sm flex items-end sm:items-center justify-center p-0 sm:p-4 opacity-0 pointer-events-none transition-all duration-300">
        <div class="bg-slate-900 border border-slate-800 w-full max-w-md rounded-t-3xl sm:rounded-3xl p-5 shadow-2xl">
            <div class="flex items-center justify-between mb-4 border-b border-slate-800 pb-3">
                <h3 class="font-black text-lg text-white flex items-center gap-2">
                    <i class="fa-solid fa-bell text-orange-500"></i>
                    הגדרות התראות פוש
                </h3>
                <button onclick="closeNotificationModal()" class="w-8 h-8 rounded-full bg-slate-800 text-slate-400 hover:text-white flex items-center justify-center">
                    <i class="fa-solid fa-xmark"></i>
                </button>
            </div>

            <div class="space-y-4 text-xs text-slate-300">
                <div class="bg-slate-950 p-4 rounded-2xl border border-slate-800 flex items-center justify-between">
                    <div>
                        <div class="font-bold text-white text-sm mb-0.5">סטטוס הרשאה בדפדפן</div>
                        <div id="permissionStatusText" class="text-orange-400 font-semibold">בודק...</div>
                    </div>
                    <button id="reqPermBtn" onclick="requestNotificationPermission()" class="bg-orange-500 hover:bg-orange-600 text-white font-bold px-3 py-2 rounded-xl text-xs">
                        אשר התראות
                    </button>
                </div>

                <div class="bg-slate-950 p-4 rounded-2xl border border-slate-800 space-y-2">
                    <div class="font-bold text-white flex items-center gap-1.5">
                        <i class="fa-solid fa-mobile-screen text-orange-400"></i>
                        טיפ חשוב לשימוש בטלפון הנייד:
                    </div>
                    <p class="text-slate-400 leading-relaxed">
                        כדי לקבל התראות פוש בצורה האמינה ביותר בסמארטפון (אנדרואיד/אייפון):
                    </p>
                    <ul class="list-disc list-inside text-slate-400 space-y-1 pr-1">
                        <li>לחץ על תפריט הדפדפן (3 נקודות / כפתור שיתוף).</li>
                        <li>בחר <span class="text-white font-bold">"הוסף למסך הבית"</span> (Add to Home Screen).</li>
                        <li>פתח את האפליקציה ישירות ממסך הבית ואשר התראות פוש!</li>
                    </ul>
                </div>

                <button onclick="triggerTestNotification()" class="w-full bg-slate-800 hover:bg-slate-700 text-orange-400 border border-orange-500/30 font-bold py-3 rounded-xl transition-all flex items-center justify-center gap-2">
                    <i class="fa-solid fa-paper-plane"></i>
                    שלח התראת בדיקה עכשיו (Test Push)
                </button>
            </div>
        </div>
    </div>

    <!-- JavaScript Logic -->
    <script>

        // --- App State ---
        let habits = [];
        let userStats = {
            xp: 0,
            level: 1,
            totalStreakRecord: 0
        };

        const BADGES = [
            { id: 'first_step', name: 'צעד ראשון', icon: 'fa-shoe-prints', req: 1, desc: 'סימנת רצף ראשון!' },
            { id: 'streak_3', name: 'להבה קטנה', icon: 'fa-fire-flame-curved', req: 3, desc: '3 ימי רצף ברציפות' },
            { id: 'streak_7', name: 'שבוע אש', icon: 'fa-fire', req: 7, desc: 'שבוע שלם של תמידות!' },
            { id: 'streak_30', name: 'אגדת הרצף', icon: 'fa-crown', req: 30, desc: '30 ימי רצף מטורפים' }
        ];

        // Load data on startup
        window.onload = function() {
            loadStoredData();
            updateStatsUI();
            renderHabits();
            renderBadges();
            checkNotificationStatus();
            initNotificationEngine();
        };

        function loadStoredData() {
            const storedHabits = localStorage.getItem('sm_habits');
            if (storedHabits) {
                try {
                    habits = JSON.parse(storedHabits);
                } catch(e) { habits = []; }
