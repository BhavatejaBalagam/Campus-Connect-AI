# Campus-Connect-AI
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Campus Connect AI</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6;
            overflow-x: hidden;
        }
        @keyframes scale-in {
            from { transform: scale(0.95); opacity: 0; }
            to { transform: scale(1); opacity: 1; }
        }
        .animate-scale-in {
            animation: scale-in 0.2s ease-out forwards;
        }
    </style>
</head>
<body class="font-inter">

    <div id="app"></div>

    <script>
        // State variables
        let currentPage = 'landing';
        let studentTab = 'portfolio';
        let adminTab = 'placements';
        let adminType = null;
        let loginError = '';
        let isEditingPortfolio = false;
        let selectedItemDetails = null; // Holds data for the details modal

        // Dummy data
        const DUMMY_PLACEMENTS = [
            { id: 1, company: 'InnovateTech', role: 'Software Engineer', location: 'Bengaluru', date: '2025-10-15', salary: '₹12 LPA' },
            { id: 2, company: 'DataGenius', role: 'Data Scientist', location: 'Hyderabad', date: '2025-11-10', salary: '₹15 LPA' },
            { id: 3, company: 'CreativeMinds', role: 'UX/UI Designer', location: 'Pune', date: '2025-10-05', salary: '₹8 LPA' },
            { id: 4, company: 'FinPulse', role: 'Financial Analyst', location: 'Mumbai', date: '2025-11-20', salary: '₹10 LPA' },
        ];

        const DUMMY_EVENTS = [
            { id: 1, type: 'hackathon', title: 'Code Fiesta 2025', details: 'A 24-hour hackathon focused on web development. Build innovative projects and compete for exciting prizes.', organizer: 'Google', date: '2025-11-20', poster: 'https://placehold.co/400x200/4F46E5/ffffff?text=Code+Fiesta+2025' },
            { id: 2, type: 'hackathon', title: 'AI for Good Challenge', details: 'Solve real-world problems using machine learning. A great opportunity to showcase your AI skills.', organizer: 'Microsoft', date: '2025-12-05', poster: 'https://placehold.co/400x200/4F46E5/ffffff?text=AI+for+Good+Challenge' },
            { id: 3, type: 'hackathon', title: 'Quantum Computing Sprint', details: 'An online sprint for quantum algorithms. Dive into the future of computing.', organizer: 'IBM', date: '2026-01-15', poster: 'https://placehold.co/400x200/4F46E5/ffffff?text=Quantum+Computing+Sprint' },
            { id: 4, type: 'internship', title: 'Full Stack Development Intern', company: 'TechSolutions Inc.', details: 'A 3-month paid internship building scalable web applications. Gain hands-on experience with modern technologies.', location: 'Remote', date: '2026-05-01', poster: 'https://placehold.co/400x200/22C55E/ffffff?text=Internship' },
            { id: 5, type: 'internship', title: 'Product Management Intern', company: 'Global Innovations', details: 'A summer internship focused on product strategy, market research, and feature definition. Perfect for aspiring product leaders.', location: 'New Delhi', date: '2026-06-15', poster: 'https://placehold.co/400x200/22C55E/ffffff?text=Internship' },
        ];

        const DUMMY_PORTFOLIOS = [
            { id: 1, name: 'Rahul Sharma', skills: ['React', 'Node.js', 'Python'], achievements: ["Top 10 Coder at Hacktoberfest", "Dean's List"], projects: ['E-commerce Platform', 'Weather App'], cgpa: '9.2' },
            { id: 2, name: 'Priya Singh', skills: ['Data Analysis', 'Machine Learning', 'SQL'], achievements: ['Published Research Paper', 'Kaggle Grandmaster'], projects: ['Customer Churn Prediction', 'Image Classifier'], cgpa: '8.8' },
            { id: 3, name: 'Amit Kumar', skills: ['Java', 'Spring Boot', 'AWS'], achievements: ['Certified AWS Cloud Practitioner', '1st Place in College Hackathon'], projects: ['Online Banking System', 'Task Management App'], cgpa: '9.5' },
        ];
        
        let placements = [...DUMMY_PLACEMENTS];
        let events = [...DUMMY_EVENTS];
        let portfolios = [...DUMMY_PORTFOLIOS];
        
        const app = document.getElementById('app');

        function render() {
            app.innerHTML = '';
            let content = '';

            switch (currentPage) {
                case 'landing':
                    content = renderLandingPage();
                    break;
                case 'student-login':
                    content = renderStudentLoginPage();
                    break;
                case 'admin-select':
                    content = renderAdminSelectPage();
                    break;
                case 'admin-login':
                    content = renderAdminLoginPage();
                    break;
                case 'student':
                    content = renderStudentDashboard();
                    break;
                case 'admin':
                    content = renderAdminDashboard();
                    break;
            }
            app.innerHTML = content + renderDetailsModal();
        }

        // --- Page and Component Rendering Functions ---
        
        function renderDetailsModal() {
            if (!selectedItemDetails) return '';

            let modalContent = '';
            // Check if the item is a placement (it won't have a 'type' property in our data)
            if (!selectedItemDetails.type) { 
                modalContent = `
                    <h3 class="text-2xl font-bold text-gray-800 mb-2">${selectedItemDetails.company}</h3>
                    <p class="text-xl text-blue-700 font-semibold mb-4">${selectedItemDetails.role}</p>
                    <div class="space-y-2 text-md text-gray-700">
                        <p><strong>💰 Salary:</strong> <span class="text-green-600 font-bold">${selectedItemDetails.salary}</span></p>
                        <p><strong>📍 Location:</strong> ${selectedItemDetails.location}</p>
                        <p><strong>🗓️ Apply By:</strong> ${new Date(selectedItemDetails.date).toLocaleDateString('en-GB', { day: 'numeric', month: 'long', year: 'numeric' })}</p>
                    </div>
                    <p class="mt-4 mb-6 bg-gray-100 p-4 rounded-lg text-gray-800">This is a premier role for aspiring professionals. Apply now to join a leading team at <strong>${selectedItemDetails.company}</strong> and accelerate your career.</p>
                    <a href="#" class="w-full text-center bg-blue-600 text-white font-bold py-3 px-6 rounded-xl shadow-lg hover:bg-blue-700 transition-colors duration-300 block">
                        Apply Now
                    </a>
                `;
            } else { // It's an event (hackathon or internship)
                modalContent = `
                    <h3 class="text-2xl font-bold text-gray-800 mb-4">${selectedItemDetails.title}</h3>
                    <p class="text-lg text-gray-600 mb-2"><strong>${selectedItemDetails.type === 'hackathon' ? 'Organizer' : '🏢 Company'}:</strong> ${selectedItemDetails.organizer || selectedItemDetails.company}</p>
                    ${selectedItemDetails.date ? `<p class="mb-2"><strong>🗓️ Date:</strong> ${new Date(selectedItemDetails.date).toLocaleDateString('en-GB', { day: 'numeric', month: 'long', year: 'numeric' })}</p>` : ''}
                    ${selectedItemDetails.location ? `<p class="mb-4"><strong>📍 Location:</strong> ${selectedItemDetails.location}</p>` : ''}
                    <p class="mb-6 bg-gray-100 p-4 rounded-lg text-gray-800">${selectedItemDetails.details}</p>
                    <a href="#" class="w-full text-center bg-indigo-600 text-white font-bold py-3 px-6 rounded-xl shadow-lg hover:bg-indigo-700 transition-colors duration-300 block">
                        ${selectedItemDetails.type === 'hackathon' ? 'Register for Hackathon' : 'Learn More & Apply'}
                    </a>
                `;
            }

            return `
                <div id="details-modal" class="fixed inset-0 bg-black bg-opacity-60 flex items-center justify-center p-4 z-50">
                    <div class="bg-white rounded-2xl shadow-2xl p-8 max-w-2xl w-full relative animate-scale-in">
                        <button data-action="close-modal" class="absolute top-4 right-4 text-gray-500 hover:text-gray-800 text-3xl font-bold">&times;</button>
                        ${modalContent}
                    </div>
                </div>
            `;
        }

        function renderLandingPage() {
            return `
                <div class="flex flex-col items-center justify-center min-h-screen bg-gray-100 p-4">
                    <div class="bg-white p-8 sm:p-12 rounded-2xl shadow-2xl max-w-lg w-full text-center transform transition-transform hover:scale-105 duration-300">
                        <h1 class="text-4xl sm:text-5xl font-extrabold text-blue-800 mb-4 tracking-tight">Campus Connect AI</h1>
                        <p class="text-xl text-gray-600 mb-10 font-medium">Your hub for career growth and opportunities.</p>
                        <div class="space-y-6">
                            <button data-page="student-login" class="w-full bg-blue-600 text-white font-bold py-3 px-6 rounded-xl shadow-lg hover:bg-blue-700 transition-colors duration-300 focus:outline-none focus:ring-4 focus:ring-blue-300">
                                Student Login
                            </button>
                            <button data-page="admin-select" class="w-full bg-indigo-600 text-white font-bold py-3 px-6 rounded-xl shadow-lg hover:bg-indigo-700 transition-colors duration-300 focus:outline-none focus:ring-4 focus:ring-indigo-300">
                                Admin Login
                            </button>
                        </div>
                    </div>
                </div>
            `;
        }

        function renderStudentLoginPage() {
            return `
                <div class="flex flex-col items-center justify-center min-h-screen bg-gray-100 p-4">
                    <div class="bg-white p-8 sm:p-12 rounded-2xl shadow-2xl max-w-lg w-full text-center">
                        <h1 class="text-4xl sm:text-5xl font-extrabold text-blue-800 mb-4 tracking-tight">Student Login</h1>
                        <form id="student-login-form" class="space-y-6">
                            <input type="text" id="student-username" placeholder="Username" class="w-full p-3 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500" required />
                            <input type="password" id="student-password" placeholder="Password" class="w-full p-3 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500" required />
                            ${loginError ? `<div class="text-red-500 text-sm">${loginError}</div>` : ''}
                            <button type="submit" class="w-full bg-blue-600 text-white font-bold py-3 px-6 rounded-xl shadow-lg hover:bg-blue-700 transition-colors duration-300">
                                Login
                            </button>
                        </form>
                        <button data-page="landing" class="mt-8 text-gray-500 hover:text-red-500 transition-colors">
                            <span class="font-medium">Go Back</span>
                        </button>
                    </div>
                </div>
            `;
        }

        function renderAdminSelectPage() {
            return `
                <div class="flex flex-col items-center justify-center min-h-screen bg-gray-100 p-4">
                    <div class="bg-white p-8 sm:p-12 rounded-2xl shadow-2xl max-w-lg w-full text-center transform transition-transform hover:scale-105 duration-300">
                        <h1 class="text-4xl sm:text-5xl font-extrabold text-indigo-800 mb-4 tracking-tight">Admin Portal</h1>
                        <p class="text-xl text-gray-600 mb-10 font-medium">Select your portal to continue.</p>
                        <div class="space-y-6">
                            <button data-admin-role="placement" class="w-full bg-indigo-600 text-white font-bold py-3 px-6 rounded-xl shadow-lg hover:bg-indigo-700 transition-colors duration-300 focus:outline-none focus:ring-4 focus:ring-indigo-300">
                                Placement Cell Portal
                            </button>
                            <button data-admin-role="club" class="w-full bg-purple-600 text-white font-bold py-3 px-6 rounded-xl shadow-lg hover:bg-purple-700 transition-colors duration-300 focus:outline-none focus:ring-4 focus:ring-purple-300">
                                Club Portal
                            </button>
                        </div>
                        <button data-page="landing" class="mt-8 text-gray-500 hover:text-red-500 transition-colors">
                            <span class="font-medium">Go Back</span>
                        </button>
                    </div>
                </div>
            `;
        }

        function renderAdminLoginPage() {
            const heading = adminType === 'placement' ? 'Placement Cell Login' : 'Club Login';
            return `
                <div class="flex flex-col items-center justify-center min-h-screen bg-gray-100 p-4">
                    <div class="bg-white p-8 sm:p-12 rounded-2xl shadow-2xl max-w-lg w-full text-center">
                        <h1 class="text-4xl sm:text-5xl font-extrabold text-indigo-800 mb-4 tracking-tight">${heading}</h1>
                        <form id="admin-login-form" class="space-y-6">
                            <input type="text" id="admin-username" placeholder="Username" class="w-full p-3 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-indigo-500" required />
                            <input type="password" id="admin-password" placeholder="Password" class="w-full p-3 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-indigo-500" required />
                            ${loginError ? `<div class="text-red-500 text-sm">${loginError}</div>` : ''}
                            <button type="submit" class="w-full bg-indigo-600 text-white font-bold py-3 px-6 rounded-xl shadow-lg hover:bg-indigo-700 transition-colors duration-300">
                                Login
                            </button>
                        </form>
                        <button data-page="admin-select" class="mt-8 text-gray-500 hover:text-red-500 transition-colors">
                            <span class="font-medium">Go Back</span>
                        </button>
                    </div>
                </div>
            `;
        }

        function renderStudentDashboard() {
            let tabContent = '';
            switch (studentTab) {
                case 'portfolio':
                    tabContent = renderMyPortfolio();
                    break;
                case 'placements':
                    tabContent = renderPlacementsPanel();
                    break;
                case 'hackathons':
                    tabContent = renderHackathonsInternshipsPanel();
                    break;
            }
            
            return `
                <div class="min-h-screen bg-gray-50 p-4 sm:p-8">
                    <header class="flex flex-col sm:flex-row items-center justify-between bg-white p-4 rounded-xl shadow-md mb-6">
                        <h1 class="text-2xl font-bold text-gray-800 mb-4 sm:mb-0">Student Dashboard</h1>
                        <button data-page="landing" class="text-gray-500 hover:text-red-500 transition-colors">
                            <span class="font-medium">Logout</span>
                        </button>
                    </header>

                    <div class="flex flex-col sm:flex-row space-y-4 sm:space-y-0 sm:space-x-4 mb-6">
                        <button data-tab="portfolio" class="w-full py-2 px-4 rounded-lg font-semibold transition-colors ${studentTab === 'portfolio' ? 'bg-blue-600 text-white shadow-md' : 'bg-white text-gray-700 hover:bg-gray-100'}">
                            My Portfolio
                        </button>
                        <button data-tab="placements" class="w-full py-2 px-4 rounded-lg font-semibold transition-colors ${studentTab === 'placements' ? 'bg-blue-600 text-white shadow-md' : 'bg-white text-gray-700 hover:bg-gray-100'}">
                            Placements
                        </button>
                        <button data-tab="hackathons" class="w-full py-2 px-4 rounded-lg font-semibold transition-colors ${studentTab === 'hackathons' ? 'bg-blue-600 text-white shadow-md' : 'bg-white text-gray-700 hover:bg-gray-100'}">
                            Hackathons & Internships
                        </button>
                    </div>
                    
                    <div class="bg-white p-6 rounded-xl shadow-md">
                        ${tabContent}
                    </div>
                </div>
            `;
        }
        
        function renderMyPortfolio() {
            const samplePortfolio = DUMMY_PORTFOLIOS[0];
            let portfolioContentHtml = '';

            if (isEditingPortfolio) {
                portfolioContentHtml = `
                    <h3 class="text-xl font-bold mb-4">Edit Portfolio</h3>
                    <form id="edit-portfolio-form" class="space-y-4">
                        <div>
                            <label for="edit-portfolio-name" class="block text-sm font-medium text-gray-700 mb-1">Name</label>
                            <input type="text" id="edit-portfolio-name" class="w-full p-3 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500" value="${samplePortfolio.name}" required />
                        </div>
                        <div>
                            <label for="edit-portfolio-skills" class="block text-sm font-medium text-gray-700 mb-1">Skills (comma separated)</label>
                            <input type="text" id="edit-portfolio-skills" class="w-full p-3 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500" value="${samplePortfolio.skills.join(', ')}" required />
                        </div>
                        <div>
                            <label for="edit-portfolio-achievements" class="block text-sm font-medium text-gray-700 mb-1">Key Achievements (comma separated)</label>
                            <textarea id="edit-portfolio-achievements" class="w-full p-3 border border-gray-300 rounded-lg resize-none h-24 focus:outline-none focus:ring-2 focus:ring-blue-500">${samplePortfolio.achievements.join(', ')}</textarea>
                        </div>
                        <div>
                            <label for="edit-portfolio-projects" class="block text-sm font-medium text-gray-700 mb-1">Notable Projects (comma separated)</label>
                            <textarea id="edit-portfolio-projects" class="w-full p-3 border border-gray-300 rounded-lg resize-none h-24 focus:outline-none focus:ring-2 focus:ring-blue-500">${samplePortfolio.projects.join(', ')}</textarea>
                        </div>
                        <div class="flex space-x-4 pt-2">
                            <button type="submit" class="w-full bg-green-600 text-white font-bold py-3 rounded-lg shadow-lg hover:bg-green-700 transition-colors">
                                Save Changes
                            </button>
                            <button type="button" data-action="cancel-edit" class="w-full bg-gray-500 text-white font-bold py-3 rounded-lg shadow-lg hover:bg-gray-600 transition-colors">
                                Cancel
                            </button>
                        </div>
                    </form>
                `;
            } else {
                portfolioContentHtml = `
                    <div class="flex justify-between items-center mb-4">
                        <h3 class="text-xl font-bold">Original Portfolio</h3>
                        <button data-action="edit-portfolio" class="bg-blue-500 text-white font-semibold py-2 px-4 rounded-lg hover:bg-blue-600 transition-colors text-sm shadow-md">
                            Edit Portfolio
                        </button>
                    </div>
                    <h3 class="text-xl font-bold mb-2">🎓 ${samplePortfolio.name}</h3>
                    <p class="mb-4"><strong>Skills:</strong> ${samplePortfolio.skills.join(', ')}</p>
                    <p class="mb-4"><strong>Key Achievements:</strong><br/>${samplePortfolio.achievements.join('<br/>')}</p>
                    <p class="mb-4"><strong>Notable Projects:</strong><br/>
                        ${samplePortfolio.projects.map(p => `<a href="#" class="text-blue-600 hover:underline" target="_blank">${p}</a>`).join(', ')}</p>
                `;
            }
            
            const aiPortfolioSection = `
                <div class="p-6 bg-gray-50 rounded-xl shadow-inner border border-gray-200 mt-6">
                    <h3 class="text-xl font-bold mb-4">AI Portfolio Builder</h3>
                    <div id="portfolio-error" class="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded-lg relative mb-4 hidden" role="alert"></div>
                    <form id="portfolio-builder-form" class="space-y-4">
                        <input type="text" id="portfolio-name" placeholder="Your Name" class="w-full p-3 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500" required />
                        <input type="text" id="portfolio-skills" placeholder="Skills (comma separated)" class="w-full p-3 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500" required />
                        <textarea id="portfolio-achievements" placeholder="Achievements" class="w-full p-3 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 resize-none h-24"></textarea>
                        <textarea id="portfolio-projects" placeholder="Projects" class="w-full p-3 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 resize-none h-24"></textarea>
                        <button type="submit" class="w-full bg-blue-600 text-white font-bold py-3 rounded-lg shadow-lg hover:bg-blue-700 transition-colors">
                            Generate Portfolio
                        </button>
                    </form>
                    <div id="portfolio-output" class="mt-8 p-6 bg-gray-100 rounded-xl shadow-inner hidden"></div>
                </div>
            `;

            return `
                <div>
                    <h2 class="text-2xl font-bold mb-4">My Portfolio</h2>
                    <div class="p-6 bg-gray-100 rounded-xl shadow-inner mb-6">
                        ${portfolioContentHtml}
                    </div>
                    ${aiPortfolioSection}
                </div>
            `;
        }
        
        function renderPlacementsPanel() {
            const searchTerm = document.getElementById('placement-search')?.value || '';
            const filteredPlacements = placements.filter(p =>
                p.company.toLowerCase().includes(searchTerm.toLowerCase()) ||
                p.role.toLowerCase().includes(searchTerm.toLowerCase()) ||
                p.location.toLowerCase().includes(searchTerm.toLowerCase())
            );

            const cardsHtml = filteredPlacements.length > 0 ?
                filteredPlacements.map(placement => `
                    <div data-action="show-details" data-item-id="${placement.id}" class="bg-gray-50 p-6 rounded-xl shadow-sm border border-gray-200 hover:shadow-lg hover:bg-gray-100 transition-all duration-300 cursor-pointer">
                        <h3 class="text-xl font-semibold text-gray-800 mb-2">${placement.company}</h3>
                        <p class="text-blue-600 font-medium mb-1">${placement.role}</p>
                        <p class="text-sm text-gray-600 mb-1">📍 ${placement.location}</p>
                        <p class="text-sm text-gray-600">💰 ${placement.salary}</p>
                    </div>
                `).join('') :
                `<p class="text-center text-gray-500 col-span-full">No placements found.</p>`;
            
            return `
                <div>
                    <h2 class="text-2xl font-bold mb-4">Available Placements</h2>
                    <input type="text" id="placement-search" placeholder="Search by company, role, or location..." value="${searchTerm}" class="w-full p-3 border rounded-lg mb-6 focus:outline-none focus:ring-2 focus:ring-blue-500" />
                    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-2 gap-6">
                        ${cardsHtml}
                    </div>
                </div>
            `;
        }

        function renderHackathonsInternshipsPanel() {
            const searchTerm = document.getElementById('events-search')?.value || '';
            const filteredEvents = events.filter(item =>
                item.title.toLowerCase().includes(searchTerm.toLowerCase()) ||
                (item.organizer && item.organizer.toLowerCase().includes(searchTerm.toLowerCase())) ||
                (item.company && item.company.toLowerCase().includes(searchTerm.toLowerCase()))
            );

            const cardsHtml = filteredEvents.length > 0 ?
                filteredEvents.map(item => `
                    <div data-action="show-details" data-item-id="${item.id}" data-item-type="event" class="bg-gray-50 p-4 rounded-xl shadow-sm border border-gray-200 hover:shadow-lg hover:bg-gray-100 transition-all duration-300 cursor-pointer flex flex-col">
                        ${item.poster ? `<img src="${item.poster}" alt="${item.title} Poster" class="w-full h-auto rounded-lg mb-4 object-cover" />` : `<div class="w-full h-32 bg-gray-200 rounded-lg mb-4 flex items-center justify-center text-gray-500 font-semibold">No Poster</div>`}
                        <h3 class="text-xl font-semibold mb-2">${item.title}</h3>
                        <p class="text-gray-600">
                            ${item.type === 'hackathon' ? `Organized by <strong>${item.organizer}</strong>` : `🏢 <strong>${item.company}</strong>`}
                        </p>
                        <p class="text-gray-600 mt-2 text-sm flex-grow">${item.details.substring(0, 70)}...</p>
                    </div>
                `).join('') :
                `<p class="text-center text-gray-500 col-span-full">No events found.</p>`;

            return `
                <div>
                    <h2 class="text-2xl font-bold mb-4">Hackathons & Internships</h2>
                    <input type="text" id="events-search" placeholder="Search by title, company, or organizer..." value="${searchTerm}" class="w-full p-3 border rounded-lg mb-6 focus:outline-none focus:ring-2 focus:ring-blue-500" />
                    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                        ${cardsHtml}
                    </div>
                </div>
            `;
        }

        // --- ADMIN SECTIONS ---
        function renderAdminDashboard() {
            let tabContent = '';
            let tabs = '';
            const isPlacementAdmin = adminType === 'placement';

            if (isPlacementAdmin) {
                tabContent = adminTab === 'placements' ? renderPlacementsManagement() :
                             adminTab === 'internships' ? renderInternshipManagement() :
                             renderPortfolioSearch();
                tabs = `
                    <button data-admin-tab="placements" class="w-full py-2 px-4 rounded-lg font-semibold transition-colors ${adminTab === 'placements' ? 'bg-indigo-600 text-white shadow-md' : 'bg-white text-gray-700 hover:bg-gray-100'}">Placements</button>
                    <button data-admin-tab="internships" class="w-full py-2 px-4 rounded-lg font-semibold transition-colors ${adminTab === 'internships' ? 'bg-indigo-600 text-white shadow-md' : 'bg-white text-gray-700 hover:bg-gray-100'}">Internships</button>
                    <button data-admin-tab="portfolios" class="w-full py-2 px-4 rounded-lg font-semibold transition-colors ${adminTab === 'portfolios' ? 'bg-indigo-600 text-white shadow-md' : 'bg-white text-gray-700 hover:bg-gray-100'}">Portfolios</button>
                `;
            } else {
                tabContent = renderHackathonsManagement();
                tabs = `<button data-admin-tab="hackathons" class="w-full py-2 px-4 rounded-lg font-semibold transition-colors ${adminTab === 'hackathons' ? 'bg-indigo-600 text-white shadow-md' : 'bg-white text-gray-700 hover:bg-gray-100'}">Hackathons</button>`;
            }
            
            return `
                <div class="min-h-screen bg-gray-50 p-4 sm:p-8">
                    <header class="flex flex-col sm:flex-row items-center justify-between bg-white p-4 rounded-xl shadow-md mb-6">
                        <h1 class="text-2xl font-bold text-gray-800 mb-4 sm:mb-0">${isPlacementAdmin ? 'Placement Cell Portal' : 'Club Portal'}</h1>
                        <button data-page="landing" class="text-gray-500 hover:text-red-500 transition-colors"><span class="font-medium">Logout</span></button>
                    </header>
                    <div class="flex flex-col sm:flex-row space-y-4 sm:space-y-0 sm:space-x-4 mb-6">${tabs}</div>
                    <div class="bg-white p-6 rounded-xl shadow-md">${tabContent}</div>
                </div>
            `;
        }
        function renderPlacementsManagement() {
             const ongoingPlacementsHtml = placements.map(p => `
                <div class="flex items-center justify-between p-4 bg-gray-100 rounded-lg shadow-sm">
                    <div><span class="font-semibold">${p.company}</span> - ${p.role}</div>
                    <button data-delete-placement-id="${p.id}" class="text-red-500 hover:text-red-700">Delete</button>
                </div>`).join('');
            return `<div>
                        <h2 class="text-2xl font-bold mb-4">Manage Placements</h2>
                        <div class="flex flex-col lg:flex-row lg:space-x-8">
                            <div class="lg:w-1/2">
                                <h3 class="text-xl font-semibold mb-4 text-gray-800">Add Placement</h3>
                                <form id="add-placement-form" class="space-y-4 mb-8">
                                    <input type="text" id="add-placement-company" placeholder="Company" class="p-3 w-full border rounded-lg" required />
                                    <input type="text" id="add-placement-role" placeholder="Role" class="p-3 w-full border rounded-lg" required />
                                    <input type="text" id="add-placement-location" placeholder="Location" class="p-3 w-full border rounded-lg" required />
                                    <input type="date" id="add-placement-date" class="p-3 w-full border rounded-lg" required />
                                    <input type="text" id="add-placement-salary" placeholder="Salary" class="p-3 w-full border rounded-lg" required />
                                    <button type="submit" class="w-full bg-indigo-600 text-white py-3 rounded-lg font-bold">Add Placement</button>
                                </form>
                            </div>
                            <div class="lg:w-1/2">
                                <h3 class="text-xl font-semibold mb-4 text-gray-800">Ongoing Placements</h3>
                                <div class="space-y-4">${ongoingPlacementsHtml}</div>
                            </div>
                        </div>
                    </div>`;
        }
        function renderInternshipManagement() {
            const internships = events.filter(e => e.type === 'internship');
            const ongoingInternshipsHtml = internships.map(h => `<div class="flex items-center justify-between p-4 bg-gray-100 rounded-lg shadow-sm"><div><span class="font-semibold">${h.title}</span> - ${h.company}</div><button data-delete-event-id="${h.id}" class="text-red-500 hover:text-red-700">Delete</button></div>`).join('');
            return `<div><h2 class="text-2xl font-bold mb-4">Manage Internships</h2><div class="flex flex-col lg:flex-row lg:space-x-8"><div class="lg:w-1/2"><h3 class="text-xl font-semibold mb-4 text-gray-800">Add Internship</h3><form id="add-internship-form" class="space-y-4 mb-8"><input type="text" id="add-internship-title" placeholder="Title" class="p-3 w-full border rounded-lg" required /><input type="text" id="add-internship-company" placeholder="Company" class="p-3 w-full border rounded-lg" required /><input type="url" id="add-internship-poster" placeholder="Poster URL" class="p-3 w-full border rounded-lg" /><textarea id="add-internship-details" placeholder="Details" class="p-3 w-full border rounded-lg h-24" required></textarea><button type="submit" class="w-full bg-indigo-600 text-white py-3 rounded-lg font-bold">Add Internship</button></form></div><div class="lg:w-1/2"><h3 class="text-xl font-semibold mb-4 text-gray-800">Ongoing Internships</h3><div class="space-y-4">${ongoingInternshipsHtml}</div></div></div></div>`;
        }
        function renderHackathonsManagement() {
            const hackathons = events.filter(e => e.type === 'hackathon');
            const ongoingHackathonsHtml = hackathons.map(h => `<div class="flex items-center justify-between p-4 bg-gray-100 rounded-lg shadow-sm"><div><span class="font-semibold">${h.title}</span> - ${h.organizer}</div><button data-delete-event-id="${h.id}" class="text-red-500 hover:text-red-700">Delete</button></div>`).join('');
            return `<div><h2 class="text-2xl font-bold mb-4">Manage Hackathons</h2><div class="flex flex-col lg:flex-row lg:space-x-8"><div class="lg:w-1/2"><h3 class="text-xl font-semibold mb-4 text-gray-800">Add Hackathon</h3><form id="add-hackathon-form" class="space-y-4 mb-8"><input type="text" id="add-hackathon-title" placeholder="Title" class="p-3 w-full border rounded-lg" required /><input type="text" id="add-hackathon-organizer" placeholder="Organizer" class="p-3 w-full border rounded-lg" required /><input type="url" id="add-hackathon-poster" placeholder="Poster URL" class="p-3 w-full border rounded-lg" /><textarea id="add-hackathon-details" placeholder="Details" class="p-3 w-full border rounded-lg h-24" required></textarea><button type="submit" class="w-full bg-indigo-600 text-white py-3 rounded-lg font-bold">Add Hackathon</button></form></div><div class="lg:w-1/2"><h3 class="text-xl font-semibold mb-4 text-gray-800">Ongoing Hackathons</h3><div class="space-y-4">${ongoingHackathonsHtml}</div></div></div></div>`;
        }
        function renderPortfolioSearch() {
            const searchTerm = document.getElementById('portfolio-search-input')?.value || '';
            const filteredPortfolios = portfolios.filter(p => p.name.toLowerCase().includes(searchTerm.toLowerCase()) || p.skills.some(skill => skill.toLowerCase().includes(searchTerm.toLowerCase())));
            const resultsHtml = filteredPortfolios.length > 0 ? filteredPortfolios.map(p => `<div class="bg-gray-100 p-6 rounded-xl shadow-inner"><h3 class="text-xl font-semibold text-gray-800">${p.name}</h3><p><strong>Skills:</strong> ${p.skills.join(', ')}</p><p><strong>CGPA:</strong> ${p.cgpa}</p></div>`).join('') : `<p class="text-center text-gray-500">No portfolios found.</p>`;
            return `<div><h2 class="text-2xl font-bold mb-4">Search Portfolios</h2><input type="text" id="portfolio-search-input" placeholder="Search by name or skills" value="${searchTerm}" class="w-full p-3 border rounded-lg mb-8" /><div class="space-y-6">${resultsHtml}</div></div>`;
        }

        // --- Event Handling and Main Logic ---

        document.addEventListener('click', (event) => {
            const target = event.target.closest('[data-page], [data-tab], [data-admin-tab], [data-admin-role], [data-delete-placement-id], [data-delete-event-id], [data-action], [data-item-id]');
            if (!target) return;

            const { page, tab, adminTab: adminTabButton, adminRole, deletePlacementId, deleteEventId, action, itemId, itemType } = target.dataset;

            if (page) {
                currentPage = page;
                loginError = '';
                isEditingPortfolio = false;
                render();
            } else if (tab) {
                studentTab = tab;
                render();
            } else if (adminTabButton) {
                adminTab = adminTabButton;
                render();
            } else if (adminRole) {
                adminType = adminRole;
                currentPage = 'admin-login';
                loginError = '';
                render();
            } else if (deletePlacementId) {
                placements = placements.filter(p => p.id !== parseInt(deletePlacementId));
                render();
            } else if (deleteEventId) {
                events = events.filter(e => e.id !== parseInt(deleteEventId));
                render();
            } else if (action === 'edit-portfolio') {
                isEditingPortfolio = true;
                render();
            } else if (action === 'cancel-edit') {
                isEditingPortfolio = false;
                render();
            } else if (action === 'show-details') {
                const id = parseInt(itemId);
                selectedItemDetails = itemType === 'event' 
                    ? events.find(e => e.id === id) 
                    : placements.find(p => p.id === id);
                render();
            } else if (action === 'close-modal') {
                selectedItemDetails = null;
                render();
            }
        });

        document.addEventListener('submit', (event) => {
            event.preventDefault();
            const formId = event.target.id;
            
            if (formId === 'student-login-form') {
                if (document.getElementById('student-username').value === 'student' && document.getElementById('student-password').value === 'password') {
                    currentPage = 'student'; loginError = '';
                } else { loginError = 'Invalid credentials. Please try again.'; }
                render();
            } else if (formId === 'admin-login-form') {
                const u = document.getElementById('admin-username').value;
                const p = document.getElementById('admin-password').value;
                let valid = (adminType === 'placement' && u === 'pc_admin' && p === 'pc_password') || (adminType === 'club' && u === 'club_admin' && p === 'club_password');
                if (valid) {
                    currentPage = 'admin';
                    adminTab = adminType === 'placement' ? 'placements' : 'hackathons';
                    loginError = '';
                } else { loginError = 'Invalid credentials. Please try again.'; }
                render();
            } else if (formId === 'add-placement-form') {
                placements.push({ id: Date.now(), company: document.getElementById('add-placement-company').value, role: document.getElementById('add-placement-role').value, location: document.getElementById('add-placement-location').value, date: document.getElementById('add-placement-date').value, salary: document.getElementById('add-placement-salary').value });
                event.target.reset();
                render();
            } else if (formId === 'add-internship-form') {
                events.push({ id: Date.now(), type: 'internship', title: document.getElementById('add-internship-title').value, company: document.getElementById('add-internship-company').value, details: document.getElementById('add-internship-details').value, poster: document.getElementById('add-internship-poster').value });
                event.target.reset();
                render();
            } else if (formId === 'add-hackathon-form') {
                events.push({ id: Date.now(), type: 'hackathon', title: document.getElementById('add-hackathon-title').value, organizer: document.getElementById('add-hackathon-organizer').value, details: document.getElementById('add-hackathon-details').value, poster: document.getElementById('add-hackathon-poster').value });
                event.target.reset();
                render();
            } else if (formId === 'edit-portfolio-form') {
                const portfolioToUpdate = DUMMY_PORTFOLIOS[0];
                portfolioToUpdate.name = document.getElementById('edit-portfolio-name').value;
                portfolioToUpdate.skills = document.getElementById('edit-portfolio-skills').value.split(',').map(s => s.trim()).filter(s => s);
                portfolioToUpdate.achievements = document.getElementById('edit-portfolio-achievements').value.split(',').map(a => a.trim()).filter(a => a);
                portfolioToUpdate.projects = document.getElementById('edit-portfolio-projects').value.split(',').map(p => p.trim()).filter(p => p);
                isEditingPortfolio = false;
                render();
            } else if (formId === 'portfolio-builder-form') {
                const name = document.getElementById('portfolio-name').value;
                const skills = document.getElementById('portfolio-skills').value;
                const achievements = document.getElementById('portfolio-achievements').value;
                const projects = document.getElementById('portfolio-projects').value;
                const errorDiv = document.getElementById('portfolio-error');
                const outputDiv = document.getElementById('portfolio-output');

                if (!name || !skills) {
                    errorDiv.textContent = "Please fill in at least Name and Skills.";
                    errorDiv.classList.remove('hidden');
                    return;
                }
                errorDiv.classList.add('hidden');
                
                const projectLinks = projects.split(',').map(p => `<a href="#" class="text-blue-600 hover:underline" target="_blank">${p.trim()}</a>`).join(', ');
                const portfolioHtml = `
                    <h3 class="text-xl font-bold mb-2">🎓 ${name}</h3>
                    <p class="mb-4"><strong>Skills:</strong> ${skills}</p>
                    <p class="mb-4"><strong>Key Achievements:</strong><br/>${achievements || 'Building expertise in chosen field'}</p>
                    <p class="mb-4"><strong>Notable Projects:</strong><br/>${projectLinks || 'Working on innovative solutions'}</p>
                    <div class="p-4 rounded-lg bg-blue-50 border-l-4 border-blue-500 text-blue-700 mt-6">
                        <h4 class="font-bold">AI-Generated Summary:</h4>
                        <p class="text-sm">Based on your profile, you demonstrate strong potential in ${skills.split(',')[0].trim()} and related technologies. Your combination of skills makes you an ideal candidate for roles in software development, data analysis, and technical innovation.</p>
                        <h4 class="font-bold mt-4">Recommended Focus Areas:</h4>
                        <ul class="list-disc list-inside text-sm">
                            <li>Continue developing expertise in ${skills.split(',')[0].trim()}</li>
                            <li>Build portfolio projects showcasing practical applications</li>
                            <li>Participate in hackathons and coding competitions</li>
                            <li>Network with industry professionals</li>
                        </ul>
                        <h4 class="font-bold mt-4">Career Readiness Score: 85/100</h4>
                    </div>
                `;
                outputDiv.innerHTML = portfolioHtml;
                outputDiv.classList.remove('hidden');
            }
        });
        
        document.addEventListener('input', (event) => {
            const id = event.target.id;
            if ((id === 'placement-search' && studentTab === 'placements') || (id === 'events-search' && studentTab === 'hackathons') || (id === 'portfolio-search-input' && adminTab === 'portfolios')) {
                render();
            }
        });

        // Initial render
        render();

    </script>
</body>
</html>

