💹 FinSmart — Financial Inclusion & Smart Financial Behaviour Analyser
A free, browser-based financial wellness platform that helps you understand your financial health, track goals, discover your money personality and get personalised advice — all without any registration or backend.

HTML5 CSS3 JavaScript No Backend localStorage SDG1 SDG8

Overview
FinSmart is a multi-page client-side web application that promotes financial awareness and stability. It calculates a Financial Stability Score, identifies your Financial Personality Type through a behavioural quiz and delivers a fully personalised action plan — with zero data ever leaving your device.

Built as part of a project supporting:

SDG 1 — No Poverty
SDG 8 — Decent Work & Economic Growth
Features
Feature	Description
Financial Stability Score	Calculates a score out of 100 based on savings rate, expense ratio and emergency fund coverage
Goal Tracker	Set financial goals with deadlines, track progress with visual bars and on-track/at-risk status
Personality Quiz	10-question behavioural quiz that identifies your financial personality type
Results & Advice	Personalised strengths, weaknesses and a 6-point action plan based on your personality
Budget Planner	Applies the 50/30/20 rule to your income and compares it with your actual spending
Debt Tracker	Track debts with interest rates and get an estimated payoff timeline and debt-free date
Score History	Line chart showing your financial health score trend over time
100% Private	All data stored in browser localStorage — nothing sent to any server
Project Structure
FinSmart/ ├── index.html # Landing page + profile setup / login ├── dashboard.html # Main hub with overview cards ├── analyzer.html # Financial input + stability score calculator ├── goals.html # Goal tracker with progress bars ├── quiz.html # 10-question personality quiz ├── results.html # Personality results + personalised advice ├── budget.html # 50/30/20 budget planner ├── debt.html # Debt tracker with payoff calculator ├── history.html # Score history with line chart ├── style.css # Shared stylesheet for all pages └── script.js # Shared JavaScript logic for all pages

Stability Score Formula
Score is calculated out of 100 points across three parameters:

Parameter	Max Points	Criteria
Savings Rate	40	≥30% = 40pts, ≥20% = 30pts, ≥10% = 20pts, >0% = 10pts
Expense Ratio	30	≤50% = 30pts, ≤60% = 22pts, ≤70% = 15pts, ≤80% = 8pts
Emergency Fund	30	≥6mo = 30pts, ≥4mo = 22pts, ≥2mo = 14pts, ≥1mo = 7pts
Health Levels: 80 – 100 → Excellent 60 – 79 → Good 40 – 59 → Fair 0 – 39 → Needs Attention

Financial Personality Types
Determined by 10 behaviour-based questions. The most frequently scored type wins.

Type	Description
The Protector	Conservative, security-focused, avoids risk, strong emergency fund habits
The Planner	Organised, goal-driven, disciplined tracker, methodical with money
The Optimizer	Growth-focused, investment-minded, comfortable with calculated risk
The Spender	Lifestyle-focused, present-oriented, needs budgeting structure
Data Storage
All data is stored locally in the browser using localStorage. No server, no database, no account required.

Key	Contents
userProfile	Name, age, occupation, city, currency, primary goal
financialData	Income, expenses, savings, emergency fund
stabilityScore	Calculated score 0–100
goals	Array of goal objects
personalityType	PR / P / O / SP
quizAnswers	Array of selected answer indices
scoreHistory	Array of score entries with date and time
debts	Array of debt objects
Tech Stack
Technology	Usage
HTML5	Structure and content across all 9 pages
CSS3	Styling, layout, colour scheme, responsive design
JavaScript (Vanilla)	All logic, calculations, quiz, charts, localStorage
Canvas API	Donut gauge chart and score history line chart
localStorage API	Client-side data persistence
Font Awesome 6	Icons throughout the application
Google Fonts (DM Sans)	Typography
No frameworks. No libraries. No backend. Pure HTML, CSS and JavaScript.

🎨 Colour Scheme
Deep Blue #0a2540 Headings and dark text Mid Blue #1a5fa8 Primary accent and links Pale Blue #daeeff Backgrounds and badges Rich Green #1a9e5c Success, savings, buttons Pale Green #d4f5e5 Tag backgrounds Background #e8f4fd Page background

SDG Alignment
SDG 1 — No Poverty FinSmart removes every barrier to financial literacy — cost, privacy risk and technical complexity — making professional-grade financial analysis available to anyone with a browser.

SDG 8 — Decent Work & Economic Growth By promoting smart financial behaviour through budgeting, saving, debt management and goal setting, FinSmart contributes to individual economic stability and growth.

Future Enhancements
Bank API Integration — Plaid / Account Aggregator (RBI) for auto data pull
Backend & Cloud Sync — Node.js + MongoDB for cross-device access
AI-Powered Advice — LLM API for conversational financial guidance
Investment Tracker — Connect to market APIs for portfolio tracking
Monthly PDF Reports — Downloadable financial health summary
Push Notifications — Goal deadline alerts and monthly reminders
Multi-Language Support — Hindi, Tamil, French, Swahili and more
Gamification — Badges, streaks and financial health levels
Family Mode — Multiple profiles for shared household planning
License
This project is open source and available under the MIT License.

Acknowledgements
Font Awesome — Icons
Google Fonts — DM Sans typography
United Nations — SDG 1 and SDG 8 framework
Built with ❤️ for Financial Inclusion Supporting SDG 1: No Poverty | SDG 8: Decent Work & Economic Growth
