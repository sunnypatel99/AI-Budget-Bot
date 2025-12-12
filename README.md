# AI Budget Bot

A personal finance automation system that processes credit card transactions from email notifications, categorizes spending using Claude AI, and generates weekly/monthly budget reports automatically delivered via Discord.

## 📋 Overview

AI Budget Bot eliminates manual finance tracking by automating the entire flow from transaction notification to budget analysis. The system processes 50+ monthly transactions with delivers actionable insights through AI-generated reports.

## ✨ Key Features

- **🤖 Automated Transaction Processing**: Monitors Gmail for credit card notifications and extracts transaction details
- **🧠 AI-Powered Categorization**: Claude AI intelligently categorizes spending across 19+ budget categories
- **📊 Smart Budget Analysis**: Weekly and monthly reports with spending variance, pace analysis, and recommendations
- **💬 Discord Integration**: Automated report delivery with real-time monitoring
- **🎯 Personalized Insights**: Direct, analytical financial recommendations without sugar-coating

## 🏗️ Architecture
```
## 🏗️ Architecture
```
┌─────────────────┐
│  Credit Card    │
│  Email Alert    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   Gmail API     │
│   (n8n Node)    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   Claude AI     │
│  Data Extract   │
└────────┬────────┘
         │
    ┌────┴────┐
    ▼         ▼
┌──────────┐ ┌─────────────┐
│ Airtable │ │  Claude AI  │
│ Database │ │   Reports   │
└──────────┘ └──────┬──────┘
                    │
              ┌─────┴─────┐
              ▼           ▼
         ┌────────┐ ┌──────────┐
         │ Weekly │ │ Monthly  │
         │ Report │ │ Report   │
         └───┬────┘ └────┬─────┘
             │           │
             └─────┬─────┘
                   ▼
             ┌──────────┐
             │ Discord  │
             │ Messages │
             └──────────┘
```
```

## 🛠️ Tech Stack

| Component | Technology |
|-----------|-----------|
| **Workflow Automation** | n8n (Cloud) |
| **AI/ML** | Claude AI (Anthropic API) |
| **Database** | Airtable |
| **Messaging** | Discord (Webhooks) |
| **Email Integration** | Gmail API (OAuth 2.0) |
| **Language** | JavaScript (n8n Code nodes) |
| **Deployment** | n8n Cloud, Scheduled Triggers |

## 📊 Workflows

### 1. Transaction Processing (Daily)
- **Trigger**: Scheduled (daily)
- **Process**: 
  - Fetches unread emails from Gmail label
  - Extracts transaction details via Claude AI
  - Categorizes spending across 19 categories
  - Stores in Airtable with proper date formatting
  - Marks emails as read

### 2. Weekly Budget Report (Every Sunday)
- **Trigger**: Scheduled (Sunday 9 AM)
- **Process**:
  - Pulls last 7 days transactions (calendar week)
  - Pulls month-to-date transactions
  - Pre-calculates spending totals by category
  - Generates AI analysis with:
    - Weekly spending vs budget
    - Month-to-date progress
    - Pace analysis (on track/overspending)
    - Notable transactions and patterns
  - Delivers formatted report to Discord

### 3. Monthly Budget Report (1st of Month)
- **Trigger**: Scheduled (1st at 9 AM)
- **Process**:
  - Analyzes entire previous month
  - Pre-calculates category totals
  - Generates comprehensive AI report with:
    - Budget performance breakdown
    - Spending patterns and insights
    - Biggest wins and problem areas
    - Actionable recommendations
  - Delivers to Discord

## 📁 Budget Structure

The system supports flexible budget allocation with customizable categories:

**Necessities:**
- Housing (Rent, Utilities, Internet)
- Food (Groceries, Dining)
- Transportation (Gas, Rideshare, Maintenance)
- Healthcare

**Discretionary Spending:**
- Shopping (Clothing, Electronics, Home Goods)
- Entertainment & Subscriptions
- Personal Care & Travel
- Education & Gifts

**Savings/Investments:**
- Can be tracked or excluded based on auto-deduction preferences

*Budget percentages and categories are fully customizable in the n8n workflows.*

## 💰 Cost Breakdown

| Service | Cost |
|---------|------|
| n8n Cloud | $24/month (2,500 executions/month) |
| Claude API | ~$0.50-2/month |
| Airtable | Free tier |
| Discord | Free |
| Gmail API | Free |
| **Total** | **~$24.50-26/month** |

## 🚀 Setup Overview

**Prerequisites:**
- n8n account (cloud or self-hosted)
- Anthropic API key
- Airtable account
- Discord server
- Gmail account with transaction notifications enabled

**High-Level Steps:**
1. Create Airtable base with required fields
2. Set up Gmail label and filters for transaction emails
3. Configure Discord webhook for reports
4. Import and configure n8n workflows
5. Set up credentials (Gmail OAuth, Anthropic API, Airtable PAT, Discord webhook)
6. Test each workflow manually
7. Activate scheduled triggers

*Detailed setup instructions available upon request.*

## 📈 Results & Impact

- **98% time reduction**: From 1 hour/week to 2 minutes/day reviewing records and reports
- **50+ transactions** processed automatically per month
- **Consistent insights** delivered without manual effort
- **$24-26/month** total operational cost

## 🎯 Key Learnings

### Technical Achievements
- Engineered multi-API integration
- Implemented AI prompt engineering to avoid LLM calculation errors (pre-calculate in code, not AI)
- Designed dynamic budget system adjusting for variable monthly income
- Built stateless weekly reports spanning month boundaries

### Best Practices Applied
- Pre-calculation of totals in code (don't rely on AI for math)
- Explicit category mapping to prevent miscategorization
- Separate housing from variable spending for accurate tracking
- Use of "Actual Spend" field for split transactions/reimbursements

## 📸 Sample Report
```
📊 Weekly Finance Report (November 30th, 2025 - December 6th, 2025

LAST 7 DAYS REVIEW
Spending Performance:
- Necessities: $108.16 / $200 budget ✅ Under by $91.84
- Guilt-free: $29.99 / $200 budget ✅ Under by $170.01

Notable Transactions: Strong grocery discipline with small Aldi/Publix trips. Mixed dining - quick bites (Taco Bell $8.03, Chick-fil-A $7.59) plus higher-end experiences. Transportation via rideshare when needed.

---

SECTION 2 - December Month-to-Date Progress
Monthly Progress:
Necessities: $56.98 vs $800 budget ($743.02 remaining)
Guilt-free: $24.00 vs $800 budget ($776.00 remaining)

Pace Analysis: 7 days in, you're significantly under pace on both categories. 
Excellent control creating buffer for holiday spending!
```

## 🔮 Future Enhancements

- [ ] Build historical trend analysis with month-over-month comparisons
- [ ] Create budget alert notifications for spending thresholds
- [ ] Add interactive dashboard with visual charts and graphs
- [ ] Implement natural language query chatbot for ad-hoc questions
- [ ] Add predictive spending forecasts using historical data

## 🤝 Contributing

This is a personal project, but suggestions and ideas are welcome! Feel free to open an issue with:
- Feature requests
- Bug reports
- Optimization ideas
- Questions about implementation

## 📄 License

MIT License - see [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Built with [n8n](https://n8n.io) workflow automation
- Powered by [Claude AI](https://anthropic.com) from Anthropic
- Inspired by the need for effortless personal finance tracking

---

**Note**: This system handles personal financial data. Always review transactions for accuracy, maintain backups, and secure your API credentials properly.

**Contact**: [Your LinkedIn](www.linkedin.com/in/sunnypatel99)
