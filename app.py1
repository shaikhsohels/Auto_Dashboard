import streamlit as st
import pandas as pd
import numpy as np
import plotly.express as px
import plotly.graph_objects as go
from plotly.subplots import make_subplots
import io
import warnings
warnings.filterwarnings('ignore')

# Page configuration
st.set_page_config(
    page_title="Auto Dashboard App",
    page_icon="📊",
    layout="wide",
    initial_sidebar_state="expanded"
)

# Custom CSS for better styling
st.markdown("""
    <style>
    .main-header {
        font-size: 3rem;
        font-weight: bold;
        text-align: center;
        color: #1f77b4;
        margin-bottom: 2rem;
    }
    .stButton>button {
        width: 100%;
        background-color: #1f77b4;
        color: white;
        font-weight: bold;
    }
    </style>
""", unsafe_allow_html=True)

def load_data(uploaded_file):
    """Load data from uploaded file (CSV, Excel, etc.)"""
    try:
        if uploaded_file.name.endswith('.csv'):
            df = pd.read_csv(uploaded_file)
        elif uploaded_file.name.endswith(('.xlsx', '.xls')):
            df = pd.read_excel(uploaded_file)
        elif uploaded_file.name.endswith('.json'):
            df = pd.read_json(uploaded_file)
        elif uploaded_file.name.endswith('.parquet'):
            df = pd.read_parquet(uploaded_file)
        else:
            st.error("Unsupported file format. Please upload CSV, Excel, JSON, or Parquet files.")
            return None
        return df
    except Exception as e:
        st.error(f"Error loading file: {str(e)}")
        return None

def detect_column_types(df):
    """Detect numeric, categorical, and datetime columns"""
    numeric_cols = df.select_dtypes(include=[np.number]).columns.tolist()
    categorical_cols = df.select_dtypes(include=['object', 'category']).columns.tolist()
    datetime_cols = df.select_dtypes(include=['datetime64']).columns.tolist()
    
    # Try to convert object columns to datetime
    for col in categorical_cols:
        try:
            pd.to_datetime(df[col], errors='raise')
            datetime_cols.append(col)
            categorical_cols.remove(col)
        except:
            pass
    
    return numeric_cols, categorical_cols, datetime_cols

def create_pie_chart(df, value_col, category_col):
    """Create a pie chart"""
    fig = px.pie(df, values=value_col, names=category_col, 
                 title=f"Pie Chart: {category_col} by {value_col}")
    return fig

def create_donut_chart(df, value_col, category_col):
    """Create a donut chart"""
    fig = px.pie(df, values=value_col, names=category_col, 
                 hole=0.4, title=f"Donut Chart: {category_col} by {value_col}")
    return fig

def create_bar_chart(df, x_col, y_col, orientation='v'):
    """Create a bar chart"""
    if orientation == 'v':
        fig = px.bar(df, x=x_col, y=y_col, title=f"Bar Chart: {y_col} by {x_col}")
    else:
        fig = px.bar(df, x=y_col, y=x_col, orientation='h', 
                    title=f"Horizontal Bar Chart: {y_col} by {x_col}")
    return fig

def create_column_chart(df, x_col, y_col):
    """Create a column chart (same as vertical bar chart)"""
    fig = px.bar(df, x=x_col, y=y_col, title=f"Column Chart: {y_col} by {x_col}")
    return fig

def create_scatter_plot(df, x_col, y_col, color_col=None, size_col=None):
    """Create a scatter plot"""
    fig = px.scatter(df, x=x_col, y=y_col, color=color_col, size=size_col,
                    title=f"Scatter Plot: {y_col} vs {x_col}")
    return fig

def create_bubble_chart(df, x_col, y_col, size_col, color_col=None):
    """Create a bubble chart"""
    fig = px.scatter(df, x=x_col, y=y_col, size=size_col, color=color_col,
                    hover_name=df.index if len(df) < 100 else None,
                    size_max=60, title=f"Bubble Chart: {y_col} vs {x_col}")
    return fig

def create_histogram(df, col, bins=30):
    """Create a histogram"""
    fig = px.histogram(df, x=col, nbins=bins, title=f"Histogram: {col}")
    return fig

def create_radial_bar_chart(df, category_col, value_col):
    """Create a radial bar chart"""
    fig = go.Figure()
    fig.add_trace(go.Barpolar(
        r=df[value_col].values,
        theta=df[category_col].values,
        name=value_col,
        marker_color='lightblue',
        marker_line_color='black',
        marker_line_width=1
    ))
    fig.update_layout(
        title=f"Radial Bar Chart: {value_col} by {category_col}",
        polar=dict(
            radialaxis=dict(visible=True),
        ),
        showlegend=False
    )
    return fig

def auto_generate_charts(df):
    """Automatically generate appropriate charts based on data"""
    numeric_cols, categorical_cols, datetime_cols = detect_column_types(df)
    
    charts = []
    
    # 1. Pie/Donut Charts - for categorical data with one numeric value
    if len(categorical_cols) >= 1 and len(numeric_cols) >= 1:
        cat_col = categorical_cols[0]
        num_col = numeric_cols[0]
        
        # Aggregate if needed
        if len(df) > 20:
            agg_df = df.groupby(cat_col)[num_col].sum().reset_index()
            agg_df = agg_df.sort_values(num_col, ascending=False).head(10)
        else:
            agg_df = df[[cat_col, num_col]].copy()
        
        charts.append(("Pie Chart", create_pie_chart(agg_df, num_col, cat_col)))
        charts.append(("Donut Chart", create_donut_chart(agg_df, num_col, cat_col)))
    
    # 2. Bar/Column Charts - categorical vs numeric
    if len(categorical_cols) >= 1 and len(numeric_cols) >= 1:
        cat_col = categorical_cols[0]
        num_col = numeric_cols[0]
        
        if len(df) > 20:
            agg_df = df.groupby(cat_col)[num_col].sum().reset_index()
            agg_df = agg_df.sort_values(num_col, ascending=False).head(15)
        else:
            agg_df = df[[cat_col, num_col]].copy()
        
        charts.append(("Bar Chart", create_bar_chart(agg_df, cat_col, num_col)))
        charts.append(("Column Chart", create_column_chart(agg_df, cat_col, num_col)))
        charts.append(("Horizontal Bar Chart", create_bar_chart(agg_df, cat_col, num_col, 'h')))
    
    # 3. Scatter Plot - two numeric columns
    if len(numeric_cols) >= 2:
        x_col, y_col = numeric_cols[0], numeric_cols[1]
        color_col = categorical_cols[0] if len(categorical_cols) >= 1 else None
        charts.append(("Scatter Plot", create_scatter_plot(df, x_col, y_col, color_col)))
    
    # 4. Bubble Chart - three numeric columns
    if len(numeric_cols) >= 3:
        x_col, y_col, size_col = numeric_cols[0], numeric_cols[1], numeric_cols[2]
        color_col = categorical_cols[0] if len(categorical_cols) >= 1 else None
        charts.append(("Bubble Chart", create_bubble_chart(df, x_col, y_col, size_col, color_col)))
    
    # 5. Histograms - for each numeric column
    for num_col in numeric_cols[:3]:  # Limit to first 3 numeric columns
        charts.append((f"Histogram: {num_col}", create_histogram(df, num_col)))
    
    # 6. Radial Bar Chart - categorical vs numeric
    if len(categorical_cols) >= 1 and len(numeric_cols) >= 1:
        cat_col = categorical_cols[0]
        num_col = numeric_cols[0]
        
        if len(df) > 20:
            agg_df = df.groupby(cat_col)[num_col].sum().reset_index()
            agg_df = agg_df.sort_values(num_col, ascending=False).head(10)
        else:
            agg_df = df[[cat_col, num_col]].copy()
        
        charts.append(("Radial Bar Chart", create_radial_bar_chart(agg_df, cat_col, num_col)))
    
    return charts

def main():
    st.markdown('<h1 class="main-header">📊 Auto Dashboard App</h1>', unsafe_allow_html=True)
    st.markdown("### Upload your data file and get instant visualizations!")
    
    # Sidebar for file upload
    with st.sidebar:
        st.header("📁 File Upload")
        uploaded_file = st.file_uploader(
            "Choose a file",
            type=['csv', 'xlsx', 'xls', 'json', 'parquet'],
            help="Upload CSV, Excel, JSON, or Parquet files"
        )
        
        if uploaded_file is not None:
            st.success(f"✅ File uploaded: {uploaded_file.name}")
            st.info(f"📏 File size: {uploaded_file.size / 1024:.2f} KB")
    
    # Main content area
    if uploaded_file is not None:
        # Load data
        df = load_data(uploaded_file)
        
        if df is not None:
            # Data overview
            st.header("📋 Data Overview")
            col1, col2, col3, col4 = st.columns(4)
            
            with col1:
                st.metric("Total Rows", len(df))
            with col2:
                st.metric("Total Columns", len(df.columns))
            with col3:
                numeric_cols, categorical_cols, datetime_cols = detect_column_types(df)
                st.metric("Numeric Columns", len(numeric_cols))
            with col4:
                st.metric("Categorical Columns", len(categorical_cols))
            
            # Display data preview
            with st.expander("👀 Preview Data", expanded=False):
                st.dataframe(df.head(20), use_container_width=True)
            
            # Display data info
            with st.expander("ℹ️ Data Information", expanded=False):
                st.write("**Column Types:**")
                st.write(df.dtypes)
                st.write("**Missing Values:**")
                missing = df.isnull().sum()
                st.write(missing[missing > 0] if missing.sum() > 0 else "No missing values!")
            
            # Auto-generate charts
            st.header("📈 Auto-Generated Dashboards")
            
            with st.spinner("🔄 Generating visualizations..."):
                charts = auto_generate_charts(df)
            
            if charts:
                st.success(f"✨ Generated {len(charts)} visualizations!")
                
                # Display charts in a grid
                num_cols = 2
                for i in range(0, len(charts), num_cols):
                    cols = st.columns(num_cols)
                    for j, (chart_name, chart_fig) in enumerate(charts[i:i+num_cols]):
                        with cols[j]:
                            st.plotly_chart(chart_fig, use_container_width=True)
            else:
                st.warning("⚠️ Could not generate charts. Please ensure your data has appropriate columns.")
            
            # Manual chart creation section
            st.header("🎨 Create Custom Charts")
            
            numeric_cols, categorical_cols, datetime_cols = detect_column_types(df)
            
            chart_type = st.selectbox(
                "Select Chart Type",
                ["Bar Chart", "Pie Chart", "Donut Chart", "Scatter Plot", 
                 "Bubble Chart", "Histogram", "Radial Bar Chart", "Column Chart"]
            )
            
            if chart_type == "Bar Chart":
                col1, col2 = st.columns(2)
                with col1:
                    x_col = st.selectbox("X-axis (Category)", categorical_cols if categorical_cols else df.columns.tolist())
                with col2:
                    y_col = st.selectbox("Y-axis (Value)", numeric_cols if numeric_cols else df.columns.tolist())
                if st.button("Generate Bar Chart"):
                    fig = create_bar_chart(df, x_col, y_col)
                    st.plotly_chart(fig, use_container_width=True)
            
            elif chart_type == "Pie Chart":
                col1, col2 = st.columns(2)
                with col1:
                    category_col = st.selectbox("Category", categorical_cols if categorical_cols else df.columns.tolist())
                with col2:
                    value_col = st.selectbox("Value", numeric_cols if numeric_cols else df.columns.tolist())
                if st.button("Generate Pie Chart"):
                    fig = create_pie_chart(df, value_col, category_col)
                    st.plotly_chart(fig, use_container_width=True)
            
            elif chart_type == "Donut Chart":
                col1, col2 = st.columns(2)
                with col1:
                    category_col = st.selectbox("Category", categorical_cols if categorical_cols else df.columns.tolist())
                with col2:
                    value_col = st.selectbox("Value", numeric_cols if numeric_cols else df.columns.tolist())
                if st.button("Generate Donut Chart"):
                    fig = create_donut_chart(df, value_col, category_col)
                    st.plotly_chart(fig, use_container_width=True)
            
            elif chart_type == "Scatter Plot":
                col1, col2 = st.columns(2)
                with col1:
                    x_col = st.selectbox("X-axis", numeric_cols if numeric_cols else df.columns.tolist())
                with col2:
                    y_col = st.selectbox("Y-axis", numeric_cols if numeric_cols else df.columns.tolist())
                color_col = st.selectbox("Color by (optional)", [None] + categorical_cols)
                if st.button("Generate Scatter Plot"):
                    fig = create_scatter_plot(df, x_col, y_col, color_col if color_col else None)
                    st.plotly_chart(fig, use_container_width=True)
            
            elif chart_type == "Bubble Chart":
                col1, col2, col3 = st.columns(3)
                with col1:
                    x_col = st.selectbox("X-axis", numeric_cols if numeric_cols else df.columns.tolist())
                with col2:
                    y_col = st.selectbox("Y-axis", numeric_cols if numeric_cols else df.columns.tolist())
                with col3:
                    size_col = st.selectbox("Size", numeric_cols if numeric_cols else df.columns.tolist())
                color_col = st.selectbox("Color by (optional)", [None] + categorical_cols)
                if st.button("Generate Bubble Chart"):
                    fig = create_bubble_chart(df, x_col, y_col, size_col, color_col if color_col else None)
                    st.plotly_chart(fig, use_container_width=True)
            
            elif chart_type == "Histogram":
                col = st.selectbox("Select Column", numeric_cols if numeric_cols else df.columns.tolist())
                bins = st.slider("Number of Bins", 10, 100, 30)
                if st.button("Generate Histogram"):
                    fig = create_histogram(df, col, bins)
                    st.plotly_chart(fig, use_container_width=True)
            
            elif chart_type == "Radial Bar Chart":
                col1, col2 = st.columns(2)
                with col1:
                    category_col = st.selectbox("Category", categorical_cols if categorical_cols else df.columns.tolist())
                with col2:
                    value_col = st.selectbox("Value", numeric_cols if numeric_cols else df.columns.tolist())
                if st.button("Generate Radial Bar Chart"):
                    fig = create_radial_bar_chart(df, category_col, value_col)
                    st.plotly_chart(fig, use_container_width=True)
            
            elif chart_type == "Column Chart":
                col1, col2 = st.columns(2)
                with col1:
                    x_col = st.selectbox("X-axis (Category)", categorical_cols if categorical_cols else df.columns.tolist())
                with col2:
                    y_col = st.selectbox("Y-axis (Value)", numeric_cols if numeric_cols else df.columns.tolist())
                if st.button("Generate Column Chart"):
                    fig = create_column_chart(df, x_col, y_col)
                    st.plotly_chart(fig, use_container_width=True)
    
    else:
        # Welcome message
        st.info("👈 Please upload a file from the sidebar to get started!")
        
        st.markdown("""
        ### 🚀 Features:
        - **Automatic Chart Generation**: Upload your data and get instant visualizations
        - **Multiple Chart Types**: Pie, Donut, Bar, Column, Scatter, Bubble, Histogram, Radial Bar
        - **Custom Charts**: Create your own visualizations with full control
        - **Multiple File Formats**: Supports CSV, Excel, JSON, and Parquet files
        - **Interactive Visualizations**: All charts are interactive using Plotly
        
        ### 📝 Supported File Formats:
        - CSV (.csv)
        - Excel (.xlsx, .xls)
        - JSON (.json)
        - Parquet (.parquet)
        """)

if __name__ == "__main__":
    main()
