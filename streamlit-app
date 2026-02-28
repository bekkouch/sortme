import streamlit as st
import pandas as pd
import io

st.set_page_config(page_title="CSV Explorer", layout="wide", page_icon="📊")

st.title("📊 CSV Explorer")
st.caption("Upload a CSV to sort, filter and explore your data")

# --- File upload ---
uploaded = st.file_uploader("Upload your CSV", type=["csv"])

if uploaded:
    # Detect separator
    sample = uploaded.read(2048).decode("utf-8-sig", errors="replace")
    uploaded.seek(0)
    sep = ";" if sample.count(";") > sample.count(",") else ","

    df = pd.read_csv(uploaded, sep=sep, encoding="utf-8-sig")
    df.columns = df.columns.str.strip()

    # Coerce numeric columns
    for col in df.columns:
        try:
            df[col] = pd.to_numeric(df[col])
        except (ValueError, TypeError):
            pass

    numeric_cols = df.select_dtypes(include="number").columns.tolist()
    text_cols = df.select_dtypes(exclude="number").columns.tolist()

    st.markdown(f"**{len(df)} rows · {len(df.columns)} columns**")

    with st.sidebar:
        st.header("🔧 Controls")

        # --- Sort ---
        st.subheader("Sort")
        sort_col = st.selectbox("Sort by", df.columns.tolist())
        sort_asc = st.radio("Order", ["⬆ Ascending", "⬇ Descending"]) == "⬆ Ascending"

        # --- Filter ---
        st.subheader("Filter")
        filters = {}

        for col in text_cols:
            unique = df[col].dropna().unique().tolist()
            if 1 < len(unique) <= 30:
                selected = st.multiselect(f"{col}", sorted([str(u) for u in unique]))
                if selected:
                    filters[col] = selected

        for col in numeric_cols:
            mn, mx = float(df[col].min()), float(df[col].max())
            if mn < mx:
                rng = st.slider(f"{col}", mn, mx, (mn, mx))
                filters[col] = rng

        # --- Column visibility ---
        st.subheader("Columns")
        visible_cols = st.multiselect("Show columns", df.columns.tolist(), default=df.columns.tolist())

    # Apply filters
    filtered = df.copy()
    for col, val in filters.items():
        if col in text_cols:
            filtered = filtered[filtered[col].astype(str).isin(val)]
        else:
            filtered = filtered[(filtered[col] >= val[0]) & (filtered[col] <= val[1])]

    # Apply sort
    try:
        filtered = filtered.sort_values(sort_col, ascending=sort_asc)
    except Exception:
        pass

    # Apply column visibility
    visible_cols = [c for c in visible_cols if c in filtered.columns]
    filtered = filtered[visible_cols] if visible_cols else filtered

    # --- Metrics row ---
    if numeric_cols:
        st.subheader("📈 Quick stats")
        metric_col = st.selectbox("Metric column", numeric_cols)
        col1, col2, col3, col4 = st.columns(4)
        col1.metric("Mean", f"{filtered[metric_col].mean():.2f}")
        col2.metric("Median", f"{filtered[metric_col].median():.2f}")
        col3.metric("Max", f"{filtered[metric_col].max():.2f}")
        col4.metric("Min", f"{filtered[metric_col].min():.2f}")

    # --- Table ---
    st.subheader(f"📋 Data ({len(filtered)} rows)")
    st.dataframe(
        filtered.reset_index(drop=True),
        use_container_width=True,
        height=500,
    )

    # --- Distribution chart ---
    if numeric_cols:
        st.subheader("📊 Distribution")
        chart_col = st.selectbox("Column to visualize", numeric_cols, key="chart")
        st.bar_chart(
            filtered[chart_col].value_counts().sort_index(),
            use_container_width=True,
        )

    # --- Download filtered result ---
    csv_bytes = filtered.to_csv(index=False).encode("utf-8-sig")
    st.download_button(
        "⬇ Download filtered CSV",
        data=csv_bytes,
        file_name="filtered_export.csv",
        mime="text/csv",
    )
else:
    st.info("👆 Upload a CSV file to get started")
    st.markdown("""
    **Features:**
    - Sort by any column (asc / desc)
    - Filter text columns by value
    - Filter numeric columns by range slider
    - Show/hide columns
    - Quick stats (mean, median, min, max)
    - Distribution chart
    - Export filtered result
    """)
