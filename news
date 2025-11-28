import streamlit as st
import pandas as pd
import requests
import json
from datetime import datetime, timedelta
import brotli
import time
import random
import re

# Page configuration
st.set_page_config(
    page_title="NSE Press Releases Dashboard",
    page_icon="📈",
    layout="wide",
    initial_sidebar_state="expanded"
)

class NSEDataExtractor:
    def __init__(self):
        self.base_url = "https://www.nseindia.com"
        self.session = requests.Session()
        self.setup_headers()
        
    def setup_headers(self):
        self.session.headers.update({
            "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36",
            "Accept": "*/*",
            "Accept-Language": "en-US,en;q=0.9",
            "Connection": "keep-alive",
            "sec-ch-ua": '"Chromium";v="142", "Google Chrome";v="142", "Not_A Brand";v="99"',
            "sec-ch-ua-mobile": "?0",
            "sec-ch-ua-platform": '"Windows"',
            "sec-fetch-dest": "empty",
            "sec-fetch-mode": "cors",
            "sec-fetch-site": "same-origin"
        })
    
    def get_fresh_cookies(self):
        try:
            response = self.session.get(self.base_url, timeout=10)
            return response.status_code == 200
        except:
            return False
    
    def fetch_press_releases(self, from_date, to_date):
        """Fetch press releases for given date range"""
        url = f"{self.base_url}/api/press-release-cms20?fromDate={from_date}&toDate={to_date}"
        
        try:
            # Get fresh cookies
            self.get_fresh_cookies()
            time.sleep(random.uniform(1, 2))
            
            # Update referer
            self.session.headers.update({
                "Referer": "https://www.nseindia.com/resources/exchange-communication-press-releases",
                "Accept-Encoding": "gzip, deflate, br"
            })
            
            # Make request
            response = self.session.get(url, timeout=15, stream=True)
            
            # Get raw content
            raw_content = b''
            for chunk in response.iter_content(chunk_size=8192):
                if chunk:
                    raw_content += chunk
            
            if response.status_code == 200:
                # Check if already decompressed
                if raw_content[:1] in (b'[', b'{'):
                    data = json.loads(raw_content.decode('utf-8'))
                    return data
                
                # Try manual decompression
                content_encoding = response.headers.get('Content-Encoding', '').lower()
                if content_encoding == 'br':
                    decompressed = brotli.decompress(raw_content)
                    data = json.loads(decompressed.decode('utf-8'))
                    return data
            
            return []
        except Exception as e:
            st.error(f"Error fetching data: {str(e)}")
            return []

def parse_press_release(item):
    """Parse press release item with enhanced attachment handling"""
    try:
        content = item.get('content', {})
        
        # Extract title
        title = content.get('title', 'No Title')
        
        # Extract body
        body = content.get('body', '')
        
        # Extract date
        date_str = content.get('field_date', item.get('changed', ''))
        date_obj = None
        timestamp = 0
        
        if date_str:
            for fmt in ['%d-%b-%Y', '%d-%m-%Y', '%Y-%m-%d', '%d/%m/%Y']:
                try:
                    date_obj = datetime.strptime(str(date_str), fmt)
                    timestamp = int(date_obj.timestamp())
                    break
                except:
                    continue
        
        if not date_obj:
            return None
            
        date = date_obj.strftime('%d-%b-%Y')
        
        # Extract category
        category = content.get('field_type', 'General')
        
        # If category field is empty, use category from nested structure
        if not category or category == "General":
            try:
                category_list = content.get('field_category_press', [])
                if category_list and len(category_list) > 0:
                    category_content = category_list[0].get('content', {})
                    category = category_content.get('name', 'General')
            except:
                pass
        
        # Extract attachment information
        attachment = None
        attachment_size = None
        
        field_attachment = content.get('field_file_attachement', {})
        if field_attachment and isinstance(field_attachment, dict):
            attachment_url = field_attachment.get('url', '')
            attachment_desc = field_attachment.get('desc', '')
            
            if attachment_url:
                attachment = {
                    'url': attachment_url,
                    'description': attachment_desc or 'Download PDF'
                }
                
                # Get file size
                size_bytes = content.get('field_file_attachement_size_bytes', '')
                if size_bytes:
                    try:
                        size_kb = int(size_bytes) / 1024
                        if size_kb > 1024:
                            attachment_size = f"{size_kb/1024:.2f} MB"
                        else:
                            attachment_size = f"{size_kb:.0f} KB"
                    except:
                        pass
        
        # Clean HTML from body
        subject = re.sub('<[^<]+?>', ' ', str(body))
        subject = ' '.join(subject.split())
        subject = subject.replace('\r', '').replace('\n', ' ').replace('\t', ' ').strip()
        
        if len(subject) > 300:
            subject = subject[:300] + "..."
        
        if not subject or subject == "...":
            subject = title or "No description available"
        
        return {
            'Date': date,
            'Subject': subject,
            'Department': category,
            'Full_Body': body,
            'Title': title,
            'Timestamp': timestamp,
            'ID': item.get('id', ''),
            'Attachment': attachment,
            'Attachment_Size': attachment_size
        }
    except Exception as e:
        import traceback
        print(f"Parse error: {str(e)}")
        print(traceback.format_exc())
        return None

# Custom CSS for attractive styling
st.markdown("""
<style>
    /* Main container styling */
    .main > div {
        padding-top: 1rem;
    }
    
    /* Header styling */
    .dashboard-header {
        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        padding: 2rem;
        border-radius: 15px;
        color: white;
        margin-bottom: 2rem;
        box-shadow: 0 8px 16px rgba(0,0,0,0.1);
    }
    
    .dashboard-title {
        font-size: 2.5rem;
        font-weight: 800;
        margin: 0;
        text-shadow: 2px 2px 4px rgba(0,0,0,0.2);
    }
    
    .dashboard-subtitle {
        font-size: 1.1rem;
        margin-top: 0.5rem;
        opacity: 0.95;
    }
    
    /* Card styling */
    .press-release-card {
        border: none;
        padding: 1.5rem;
        border-radius: 12px;
        margin-bottom: 1.5rem;
        background: white;
        box-shadow: 0 4px 12px rgba(0,0,0,0.08);
        transition: all 0.3s ease;
        border-left: 4px solid #667eea;
    }
    
    .press-release-card:hover {
        box-shadow: 0 8px 24px rgba(0,0,0,0.12);
        transform: translateY(-4px);
    }
    
    /* Badge styling */
    .date-badge {
        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        color: white;
        padding: 6px 14px;
        border-radius: 20px;
        font-size: 13px;
        font-weight: 600;
        display: inline-block;
        box-shadow: 0 2px 8px rgba(102, 126, 234, 0.3);
    }
    
    .category-badge {
        background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
        color: white;
        padding: 6px 14px;
        border-radius: 20px;
        font-size: 12px;
        font-weight: 600;
        margin-left: 8px;
        display: inline-block;
        box-shadow: 0 2px 8px rgba(245, 87, 108, 0.3);
    }
    
    .attachment-badge {
        background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
        color: white;
        padding: 6px 14px;
        border-radius: 20px;
        font-size: 11px;
        font-weight: 600;
        margin-left: 8px;
        display: inline-block;
        box-shadow: 0 2px 8px rgba(79, 172, 254, 0.3);
    }
    
    /* Card title */
    .card-title {
        font-size: 1.1rem;
        font-weight: 700;
        color: #2d3748;
        margin-top: 1rem;
        margin-bottom: 0.8rem;
        line-height: 1.5;
    }
    
    /* Attachment link styling */
    .attachment-link {
        display: inline-flex;
        align-items: center;
        padding: 10px 20px;
        background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
        color: white;
        text-decoration: none;
        border-radius: 8px;
        font-weight: 600;
        font-size: 14px;
        margin-top: 10px;
        transition: all 0.3s ease;
        box-shadow: 0 3px 10px rgba(79, 172, 254, 0.3);
    }
    
    .attachment-link:hover {
        transform: translateY(-2px);
        box-shadow: 0 5px 15px rgba(79, 172, 254, 0.4);
        text-decoration: none;
        color: white;
    }
    
    /* Metrics styling */
    .metric-card {
        background: white;
        padding: 1rem;
        border-radius: 10px;
        box-shadow: 0 2px 8px rgba(0,0,0,0.06);
        text-align: center;
    }
    
    /* Sidebar styling */
    .css-1d391kg {
        background: #f7fafc;
    }
    
    /* Button styling */
    .stButton button {
        border-radius: 8px;
        font-weight: 600;
        transition: all 0.3s ease;
    }
    
    .stButton button:hover {
        transform: translateY(-2px);
        box-shadow: 0 4px 12px rgba(0,0,0,0.15);
    }
    
    /* Divider */
    hr {
        margin: 2rem 0;
        border: none;
        height: 1px;
        background: linear-gradient(90deg, transparent, #cbd5e0, transparent);
    }
    
    /* Expander styling */
    .streamlit-expanderHeader {
        background-color: #f7fafc;
        border-radius: 8px;
        font-weight: 600;
    }
    
    /* Info boxes */
    .stAlert {
        border-radius: 10px;
        border: none;
        box-shadow: 0 2px 8px rgba(0,0,0,0.06);
    }
</style>
""", unsafe_allow_html=True)

# Initialize session state
if 'data' not in st.session_state:
    st.session_state.data = None
if 'last_fetch' not in st.session_state:
    st.session_state.last_fetch = None
if 'from_date' not in st.session_state:
    st.session_state.from_date = datetime.now() - timedelta(days=7)
if 'to_date' not in st.session_state:
    st.session_state.to_date = datetime.now()

# Dashboard Header
st.markdown("""
<div class="dashboard-header">
    <h1 class="dashboard-title">📈 NSE Press Releases Dashboard</h1>
    <p class="dashboard-subtitle">Stay informed with the latest updates from National Stock Exchange of India</p>
</div>
""", unsafe_allow_html=True)

# Sidebar filters
st.sidebar.header("🔍 Filter & Search Options")

# Quick date range buttons
st.sidebar.subheader("⚡ Quick Date Range")
period_cols = st.sidebar.columns(3)

with period_cols[0]:
    if st.button("Today", use_container_width=True, key="1d"):
        st.session_state.to_date = datetime.now()
        st.session_state.from_date = st.session_state.to_date - timedelta(days=1)
        st.rerun()
    if st.button("3 Months", use_container_width=True, key="3m"):
        st.session_state.to_date = datetime.now()
        st.session_state.from_date = st.session_state.to_date - timedelta(days=90)
        st.rerun()
with period_cols[1]:
    if st.button("1 Week", use_container_width=True, key="1w"):
        st.session_state.to_date = datetime.now()
        st.session_state.from_date = st.session_state.to_date - timedelta(weeks=1)
        st.rerun()
    if st.button("6 Months", use_container_width=True, key="6m"):
        st.session_state.to_date = datetime.now()
        st.session_state.from_date = st.session_state.to_date - timedelta(days=180)
        st.rerun()
with period_cols[2]:
    if st.button("1 Month", use_container_width=True, key="1m"):
        st.session_state.to_date = datetime.now()
        st.session_state.from_date = st.session_state.to_date - timedelta(days=30)
        st.rerun()
    if st.button("1 Year", use_container_width=True, key="1y"):
        st.session_state.to_date = datetime.now()
        st.session_state.from_date = st.session_state.to_date - timedelta(days=365)
        st.rerun()

# Custom date range
st.sidebar.subheader("📅 Custom Date Range")
from_date_input = st.sidebar.date_input("From Date", st.session_state.from_date)
to_date_input = st.sidebar.date_input("To Date", st.session_state.to_date)

# Update session state with manual date input
st.session_state.from_date = from_date_input
st.session_state.to_date = to_date_input

# Fetch data button
if st.sidebar.button("🔄 Fetch Data", type="primary", use_container_width=True):
    with st.spinner("🔍 Fetching press releases from NSE India..."):
        extractor = NSEDataExtractor()
        from_date_str = from_date_input.strftime('%d-%m-%Y')
        to_date_str = to_date_input.strftime('%d-%m-%Y')
        
        st.sidebar.info(f"📅 Period: {from_date_str} to {to_date_str}")
        
        raw_data = extractor.fetch_press_releases(from_date_str, to_date_str)
        
        if raw_data and len(raw_data) > 0:
            # Parse data
            parsed_data = []
            parse_errors = 0
            
            progress_bar = st.sidebar.progress(0)
            status_text = st.sidebar.empty()
            
            for idx, item in enumerate(raw_data):
                status_text.text(f"Processing {idx+1}/{len(raw_data)}...")
                progress_bar.progress((idx + 1) / len(raw_data))
                
                parsed = parse_press_release(item)
                if parsed:
                    parsed_data.append(parsed)
                else:
                    parse_errors += 1
            
            progress_bar.empty()
            status_text.empty()
            
            # Show parsing results
            if parsed_data:
                st.session_state.data = pd.DataFrame(parsed_data)
                st.session_state.last_fetch = datetime.now()
                
                col1, col2 = st.sidebar.columns(2)
                with col1:
                    st.success(f"✅ Loaded: {len(parsed_data)}")
                with col2:
                    if parse_errors > 0:
                        st.warning(f"⚠️ Skipped: {parse_errors}")
                
                st.sidebar.success("🎉 Data loaded successfully!")
            else:
                st.sidebar.error("❌ Failed to parse data. Please try again.")
        else:
            st.sidebar.error("❌ No data available for selected date range.")

# Category filter
selected_category = "All"
search_term = ""
show_attachments_only = False

if st.session_state.data is not None and len(st.session_state.data) > 0:
    st.sidebar.divider()
    
    # Category filter
    st.sidebar.subheader("🏢 Filter by Department")
    categories = ["All"] + sorted(st.session_state.data['Department'].unique().tolist())
    selected_category = st.sidebar.selectbox("Select Department", categories, label_visibility="collapsed")
    
    # Attachment filter
    st.sidebar.subheader("📎 Attachments")
    show_attachments_only = st.sidebar.checkbox("Show only items with attachments")
    
    # Search box
    st.sidebar.subheader("🔎 Search")
    search_term = st.sidebar.text_input("Search keywords", "", label_visibility="collapsed", 
                                        placeholder="Enter keywords to search...")
    
    # Clear filters
    if st.sidebar.button("🗑️ Clear All Filters", use_container_width=True):
        st.rerun()

# Main content
if st.session_state.data is not None and len(st.session_state.data) > 0:
    df = st.session_state.data.copy()
    
    # Apply filters
    if selected_category != "All":
        df = df[df['Department'] == selected_category]
    
    if show_attachments_only:
        df = df[df['Attachment'].notna()]
    
    if search_term:
        df = df[df['Subject'].str.contains(search_term, case=False, na=False) | 
                df['Title'].str.contains(search_term, case=False, na=False)]
    
    # Sort by date (most recent first)
    df = df.sort_values('Timestamp', ascending=False)
    
    # Display metrics bar
    col1, col2, col3, col4, col5 = st.columns([2, 1, 1, 1, 1])
    
    with col1:
        st.markdown(f"""
        <div class="metric-card">
            <div style="font-size: 0.9rem; color: #718096; font-weight: 600;">PERIOD</div>
            <div style="font-size: 0.95rem; color: #2d3748; font-weight: 700; margin-top: 4px;">
                {from_date_input.strftime('%d %b')} - {to_date_input.strftime('%d %b %Y')}
            </div>
        </div>
        """, unsafe_allow_html=True)
    
    with col2:
        st.metric("📊 Total", len(df))
    
    with col3:
        attachments_count = df['Attachment'].notna().sum()
        st.metric("📎 Files", attachments_count)
    
    with col4:
        # Download CSV
        csv = df[['Date', 'Title', 'Department', 'Subject']].to_csv(index=False)
        st.download_button(
            label="📥 Export",
            data=csv,
            file_name=f"nse_releases_{datetime.now().strftime('%Y%m%d')}.csv",
            mime="text/csv",
            use_container_width=True
        )
    
    with col5:
        view_type = st.selectbox("View Mode", ["📋 Cards", "📄 List"], label_visibility="collapsed")
    
    st.divider()
    
    # Display data
    if view_type == "📋 Cards":
        # Card view with 2 columns
        if len(df) == 0:
            st.info("🔍 No press releases found matching your filters. Try adjusting the filters.")
        else:
            cols = st.columns(2)
            for idx, row in df.iterrows():
                with cols[idx % 2]:
                    # Build attachment badge
                    attachment_html = ""
                    if row['Attachment']:
                        size_text = f" • {row['Attachment_Size']}" if row['Attachment_Size'] else ""
                        attachment_html = f'<span class="attachment-badge">📎 PDF{size_text}</span>'
                    
                    st.markdown(f"""
                    <div class="press-release-card">
                        <div>
                            <span class="date-badge">📅 {row['Date']}</span>
                            <span class="category-badge">{row['Department']}</span>
                            {attachment_html}
                        </div>
                        <div class="card-title">{row['Title']}</div>
                        <div style="color: #718096; font-size: 0.9rem; line-height: 1.6;">
                            {row['Subject'][:180]}{'...' if len(row['Subject']) > 180 else ''}
                        </div>
                    </div>
                    """, unsafe_allow_html=True)
                    
                    # Expandable details with attachment link
                    with st.expander("📖 Read Full Details"):
                        st.markdown(row['Full_Body'], unsafe_allow_html=True)
                        
                        if row['Attachment']:
                            st.markdown("---")
                            st.markdown(f"""
                            <a href="{row['Attachment']['url']}" target="_blank" class="attachment-link">
                                📄 {row['Attachment']['description']}
                                {' (' + row['Attachment_Size'] + ')' if row['Attachment_Size'] else ''}
                            </a>
                            """, unsafe_allow_html=True)
    
    else:
        # List view
        if len(df) == 0:
            st.info("🔍 No press releases found matching your filters. Try adjusting the filters.")
        else:
            for idx, row in df.iterrows():
                # Build badges
                attachment_badge = ""
                if row['Attachment']:
                    size_text = f" • {row['Attachment_Size']}" if row['Attachment_Size'] else ""
                    attachment_badge = f'<span class="attachment-badge">📎 PDF{size_text}</span>'
                
                header_html = f"""
                <div style="margin-bottom: 10px;">
                    <span class="date-badge">📅 {row['Date']}</span>
                    <span class="category-badge">{row['Department']}</span>
                    {attachment_badge}
                </div>
                <div style="font-size: 1.1rem; font-weight: 600; color: #2d3748;">{row['Title']}</div>
                """
                
                with st.expander(header_html, expanded=False):
                    st.markdown(f"**Subject:** {row['Subject']}")
                    st.divider()
                    st.markdown(row['Full_Body'], unsafe_allow_html=True)
                    
                    if row['Attachment']:
                        st.markdown("---")
                        col1, col2 = st.columns([3, 1])
                        with col1:
                            st.markdown(f"""
                            <a href="{row['Attachment']['url']}" target="_blank" class="attachment-link">
                                📄 Download {row['Attachment']['description']}
                                {' (' + row['Attachment_Size'] + ')' if row['Attachment_Size'] else ''}
                            </a>
                            """, unsafe_allow_html=True)

else:
    # Welcome screen
    st.markdown("""
    <div style="background: white; padding: 2rem; border-radius: 12px; box-shadow: 0 4px 12px rgba(0,0,0,0.08); margin-bottom: 2rem;">
        <h3 style="color: #2d3748; margin-top: 0;">👋 Welcome to NSE Press Releases Dashboard</h3>
        <p style="color: #718096; font-size: 1.1rem;">
            Get started by clicking the <strong>"🔄 Fetch Data"</strong> button in the sidebar to load the latest press releases.
        </p>
    </div>
    """, unsafe_allow_html=True)
    
    col1, col2 = st.columns(2)
    
    with col1:
        st.markdown("""
        ### 📋 Quick Start Guide:
        
        **1️⃣ Select Date Range**
        - Use quick buttons (Today, 1 Week, 1 Month, etc.)
        - Or choose custom dates
        
        **2️⃣ Fetch Data**
        - Click the **"🔄 Fetch Data"** button
        - Wait for data to load
        
        **3️⃣ Filter & Search**
        - Filter by department
        - Show only items with attachments
        - Search by keywords
        
        **4️⃣ View & Download**
        - Switch between Card and List views
        - Download PDF attachments
        - Export data as CSV
        """)
    
    with col2:
        st.markdown("""
        ### 📂 Available Categories:
        
        - 🔍 **Surveillance** - Market monitoring
        - 📊 **NSE Indices** - Index updates
        - 🏢 **NSE Listing** - New listings
        - 💼 **Corporate Communications**
        - 🏦 **NSE Clearing** - Settlement updates
        - ✅ **Member Compliance**
        - 👥 **Investor Services Cell**
        - 📢 **General** - Other announcements
        
        ### ✨ Features:
        - 📎 Direct PDF downloads
        - 🔎 Advanced filtering
        - 📥 CSV export
        - 📱 Responsive design
        """)

# Footer
st.divider()
footer_cols = st.columns([2, 2, 1])

with footer_cols[0]:
    if st.session_state.last_fetch:
        st.caption(f"⏰ Last updated: {st.session_state.last_fetch.strftime('%d-%b-%Y %I:%M:%S %p')}")
    else:
        st.caption("⏰ No data loaded yet")

with footer_cols[1]:
    st.caption("📡 Data source: NSE India Official API")

with footer_cols[2]:
    if st.session_state.data is not None:
        st.caption(f"💾 {len(st.session_state.data)} total records")
