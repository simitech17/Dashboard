import streamlit as st
import feedparser
import pandas as pd
import json
import os
from datetime import datetime
import ssl
import time

# SSL fallback (keep if you need it)
if hasattr(ssl, "_create_unverified_context"):
    ssl._create_default_https_context = ssl._create_unverified_context

USER_AGENT = (
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) "
    "AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
)

SEEN_FILE = "seen.json"

DEFAULT_FEEDS = """# Market / business news (free)
https://feeds.bbci.co.uk/news/business/rss.xml
https://feeds.bbci.co.uk/news/technology/rss.xml
https://feeds.npr.org/1006/rss.xml
https://www.aljazeera.com/xml/rss/all.xml
https://feeds.content.dowjones.io/public/rss/mw_realtimeheadlines
https://feeds.content.dowjones.io/public/rss/mw_topstories
https://feeds.content.dowjones.io/public/rss/mw_marketpulse
https://feeds.content.dowjones.io/public/rss/mw_bulletins
https://www.sec.gov/news/pressreleases.rss
https://www.sec.gov/news/speeches-statements.rss
https://www.sec.gov/enforcement-litigation/litigation-releases/rss
https://www.sec.gov/enforcement-litigation/administrative-proceedings/rss
https://www.sec.gov/enforcement-litigation/trading-suspensions/rss
https://www.federalreserve.gov/feeds/press_all.xml
https://www.federalreserve.gov/feeds/press_monetary.xml
https://www.federalreserve.gov/feeds/press_bcreg.xml
https://www.federalreserve.gov/feeds/press_nr.xml
https://www.federalreserve.gov/feeds/speeches.xml
"""

DEFAULT_WATCHLIST = """M&A
distress
AI
transaction
technology
innovation
law
corporate law
corporation
"""
SOURCE_WEIGHTS = {
    "sec.gov": 3.0,
    "federalreserve.gov": 2.5,
    "feeds.content.dowjones.io": 1.6,   # MarketWatch feeds
    "feeds.bbci.co.uk": 1.2,
    "feeds.npr.org": 1.1,
    "aljazeera.com": 1.0,
}
DEFAULT_SOURCE_WEIGHT = 1.0

def source_weight(feed_url: str) -> float:
    for domain, w in SOURCE_WEIGHTS.items():
        if domain in feed_url:
            return w
    return DEFAULT_SOURCE_WEIGHT

def load_seen() -> set[str]:
    if os.path.exists(SEEN_FILE):
        with open(SEEN_FILE, "r", encoding="utf-8") as f:
            return set(json.load(f))
    return set()

SAVE_FILE = "saved.json"

def load_saved() -> list[dict]:
    if os.path.exists(SAVE_FILE):
        with open(SAVE_FILE, "r", encoding="utf-8") as f:
            return json.load(f)
    return []

def save_saved(items: list[dict]) -> None:
    with open(SAVE_FILE, "w", encoding="utf-8") as f:
        json.dump(items, f, ensure_ascii=False, indent=2)

def save_seen(seen: set[str]) -> None:
    with open(SEEN_FILE, "w", encoding="utf-8") as f:
        json.dump(sorted(seen), f, ensure_ascii=False, indent=2)

def parse_lines(block: str) -> list[str]:
    lines = []
    for line in block.splitlines():
        line = line.strip()
        if not line:
            continue
        if line.startswith("#"):
            continue
        lines.append(line)
    return lines

def score(text: str, watchlist: list[str]) -> tuple[int, list[str]]:
    hits = []
    s = 0
    lower = text.lower()
    for kw in watchlist:
        if kw.lower() in lower:
            hits.append(kw)
            s += 15
    return min(s, 100), hits

def entry_time(entry) -> datetime | None:
    t = entry.get("published_parsed") or entry.get("updated_parsed")
    if not t:
        return None
    return datetime(*t[:6])

import re

def first_sentence(html_or_text: str, max_len: int = 220) -> str:
    if not html_or_text:
        return ""
    # crude HTML strip + whitespace normalize
    text = re.sub(r"<[^>]+>", " ", html_or_text)
    text = re.sub(r"\s+", " ", text).strip()
    if not text:
        return ""
    parts = re.split(r"(?<=[.!?])\s+", text)
    s = parts[0] if parts else text
    return (s[: max_len - 1] + "…") if len(s) > max_len else s

def build_exec_summary(df_view: pd.DataFrame) -> str:
    if df_view is None or df_view.empty:
        return "No items available for summary."

    top = df_view.sort_values(["score", "published"], ascending=[False, False]).head(10).copy()

    # Theme counts from hits
    hit_counts = {}
    for h in top.get("hits", []):
        for kw in str(h).split(","):
            kw = kw.strip()
            if kw:
                hit_counts[kw] = hit_counts.get(kw, 0) + 1
    top_themes = sorted(hit_counts.items(), key=lambda x: x[1], reverse=True)[:6]

    lines = []
    lines.append("### Executive Summary (Top 10)")
    lines.append(f"**Coverage:** {len(top)} priority items • **Max score:** {int(top['score'].max())}")
    if top_themes:
        lines.append("**Dominant themes:** " + ", ".join([f"{k} ({v})" for k, v in top_themes]))
    lines.append("")

    lines.append("#### Key Takeaways")
    for _, r in top.iterrows():
        snippet = first_sentence(r.get("summary", ""))
        lines.append(
            f"- **[{int(r['score'])}] {r.get('title','')}**  \n"
            f"  _{r.get('published','')} • {r.get('source','')}_  \n"
            f"  {snippet}  \n"
            f"  {r.get('link','')}"
        )

    return "\n".join(lines)

def run_app():
    st.set_page_config(page_title="Corporate News Dashboard", layout="wide")

    # --- Load latest results safely ---
    df = st.session_state.get("result_df")
    if df is None:
        df = pd.DataFrame(columns=["score", "source", "title", "hits", "published", "link", "select"])

    # --- CSS (visual polish) ---
    st.markdown("""
    <style>
    .block-container { padding-top: 1.2rem; padding-bottom: 3rem; }
    div[data-testid="stSidebar"] { border-right: 1px solid rgba(0,0,0,.06); }
    </style>
    """, unsafe_allow_html=True)

    st.title("Corporate News Dashboard")
    st.caption("Rudimental M&A  signals dashboard made by Simone Colavecchia")
    st.divider()

    # --- KPI row (must be normal Python, not inside markdown) ---
    top_score = int(df["score"].max()) if (not df.empty and "score" in df.columns) else 0
    sources = int(df["source"].nunique()) if (not df.empty and "source" in df.columns) else 0

    c1, c2, c3, c4 = st.columns(4)
    c1.metric("Signals", len(df))
    c2.metric("Top score", top_score)
    c3.metric("Sources", sources)
    c4.metric("Archived", len(load_saved()))
    st.divider()

    # --- Sidebar (all inside run_app) ---
    with st.sidebar:
        st.header("Configuration")

        feeds_text = st.text_area("RSS feeds (one per line)", value=DEFAULT_FEEDS, height=220)
        watchlist_text = st.text_area("Watchlist keywords (one per line)", value=DEFAULT_WATCHLIST, height=180)

        threshold = st.slider("Signal threshold", 0, 100, 15, 5)
        max_items = st.slider("Max items per feed", 10, 100, 40, 10)
        only_new = st.checkbox("Only show NEW items (uses seen.json)", value=True)

        colA, colB = st.columns(2)
        run = colA.button("Fetch signals")
        reset = colB.button("Reset seen list")

    if reset:
        if os.path.exists(SEEN_FILE):
            os.remove(SEEN_FILE)
        st.success("Seen list reset. Next run will treat all items as new.")

    # --- Tabs ---
    tab_feed, tab_saved = st.tabs(["📡 Source Feed", "💾 Saved Archive"])

    # ---------------- TAB 1: FEED ----------------
    with tab_feed:
        if run:
            feeds = parse_lines(feeds_text)
            watchlist = parse_lines(watchlist_text)

            seen = load_seen() if only_new else set()
            rows = []

            with st.spinner("Pulling feeds and scoring items..."):
                for url in feeds:
                    feed = feedparser.parse(url, agent=USER_AGENT)
                    for e in feed.entries[:max_items]:
                        link = e.get("link")
                        if not link:
                            continue
                        if only_new and link in seen:
                            continue

                        title = e.get("title", "")
                        summary = e.get("summary", "")
                        text = f"{title}\n{summary}"

                        base_score, hits = score(text, watchlist)
                        w = source_weight(url)
                        weighted_score = min(int(base_score * w), 100)

                        if weighted_score >= threshold:
                            dt = entry_time(e)
                            rows.append({
                                "select": False,
                                "score": weighted_score,
                                "title": title,
                                "hits": ", ".join(hits),
                                "published": dt.isoformat(sep=" ", timespec="minutes") if dt else "",
                                "link": link,
                                "source": url,
                            })

                        if only_new:
                            seen.add(link)

            if only_new:
                save_seen(seen)

            if rows:
                df = pd.DataFrame(rows).sort_values(by=["score", "published"], ascending=[False, False])
                st.session_state["result_df"] = df
            else:
                st.session_state["result_df"] = None
                st.info("No matching items. Lower threshold or broaden your watchlist.")

        # reload df after fetch (important)
        df = st.session_state.get("result_df")
        if df is None or df.empty:
            st.write("Click **Fetch signals** to load news.")
            return

        # --- Filters (nice UX) ---
        with st.expander("Filters", expanded=True):
            min_score = st.slider("Min score", 0, 100, 30, 5)
            source_filter = st.multiselect("Source", sorted(df["source"].unique()))
            q = st.text_input("Search title/hits")

        view = df.copy()
        view = view[view["score"] >= min_score]
        if source_filter:
            view = view[view["source"].isin(source_filter)]
        if q:
            ql = q.lower()
            view = view[
                view["title"].str.lower().str.contains(ql, na=False)
                | view["hits"].str.lower().str.contains(ql, na=False)
            ]

        st.caption("Select rows to archive them.")

        edited_df = st.data_editor(
            view,
            column_config={
                "select": st.column_config.CheckboxColumn("Archive", help="Check to save", default=False),
                "link": st.column_config.LinkColumn("Link", display_text="Open"),
                "score": st.column_config.NumberColumn("Score", format="%d"),
            },
            disabled=["score", "title", "hits", "published", "link", "source"],
            hide_index=True,
            use_container_width=True,
            key="feed_editor",
        )

        # Auto-save selected
        selected_rows = edited_df[edited_df["select"] == True]
        if not selected_rows.empty:
            saved_items = load_saved()
            existing_links = {x.get("link") for x in saved_items}

            new_saves = 0
            for _, row in selected_rows.iterrows():
                if row["link"] not in existing_links:
                    saved_items.append({
                        "saved_at": datetime.now().isoformat(timespec="seconds"),
                        "title": row["title"],
                        "link": row["link"],
                        "hits": row["hits"],
                        "published": row["published"],
                        "score": int(row["score"]),
                        "source": row["source"],
                    })
                    existing_links.add(row["link"])
                    new_saves += 1

            if new_saves:
                save_saved(saved_items)
                st.toast(f"✅ Archived {new_saves} items!")

    # ---------------- TAB 2: SAVED ----------------
    with tab_saved:
        saved_data = load_saved()
        if not saved_data:
            st.info("Archive is empty. Save items from the Feed tab.")
            return

        saved_df = pd.DataFrame(saved_data).sort_values("saved_at", ascending=False)
        st.write(f"**Total Saved:** {len(saved_df)}")

        st.dataframe(
            saved_df,
            column_config={"link": st.column_config.LinkColumn("Link", display_text="Open")},
            use_container_width=True,
            hide_index=True
        )

        if st.button("🗑️ Clear Archive"):
            if os.path.exists(SAVE_FILE):
                os.remove(SAVE_FILE)
            st.rerun()

def run_app():
    st.set_page_config(page_title="Corporate News Dashboard", layout="wide")

    # --- Load latest results safely ---
    df = st.session_state.get("result_df")
    if df is None:
        df = pd.DataFrame(columns=["score", "source", "title", "hits", "published", "link", "select"])

    # --- CSS (visual polish) ---
    st.markdown("""
    <style>
    .block-container { padding-top: 1.2rem; padding-bottom: 3rem; }
    div[data-testid="stSidebar"] { border-right: 1px solid rgba(0,0,0,.06); }
    </style>
    """, unsafe_allow_html=True)

    st.title("Corporate News Dashboard")
    st.caption("M&A signals dashboard made by Simone Colavecchia")
    st.divider()

    # --- KPI row (must be normal Python, not inside markdown) ---
    top_score = int(df["score"].max()) if (not df.empty and "score" in df.columns) else 0
    sources = int(df["source"].nunique()) if (not df.empty and "source" in df.columns) else 0

    c1, c2, c3, c4 = st.columns(4)
    c1.metric("Signals", len(df))
    c2.metric("Top score", top_score)
    c3.metric("Sources", sources)
    c4.metric("Archived", len(load_saved()))
    st.divider()

    # --- Sidebar (all inside run_app) ---
    with st.sidebar:
        st.header("Configuration")

        feeds_text = st.text_area("RSS feeds (one per line)", value=DEFAULT_FEEDS, height=220)
        watchlist_text = st.text_area("Watchlist keywords (one per line)", value=DEFAULT_WATCHLIST, height=180)

        threshold = st.slider("Signal threshold", 0, 100, 15, 5)
        max_items = st.slider("Max items per feed", 10, 100, 40, 10)
        only_new = st.checkbox("Only show NEW items (uses seen.json)", value=True)

        colA, colB = st.columns(2)
        run = colA.button("Fetch signals")
        reset = colB.button("Reset seen list")

    if reset:
        if os.path.exists(SEEN_FILE):
            os.remove(SEEN_FILE)
        st.success("Seen list reset. Next run will treat all items as new.")

    # --- Tabs ---
    tab_feed, tab_saved = st.tabs(["📡 Source Feed", "💾 Saved Archive"])

    # ---------------- TAB 1: FEED ----------------
    with tab_feed:
        if run:
            feeds = parse_lines(feeds_text)
            watchlist = parse_lines(watchlist_text)

            seen = load_seen() if only_new else set()
            rows = []

            with st.spinner("Pulling feeds and scoring items..."):
                for url in feeds:
                    feed = feedparser.parse(url, agent=USER_AGENT)
                    for e in feed.entries[:max_items]:
                        link = e.get("link")
                        if not link:
                            continue
                        if only_new and link in seen:
                            continue

                        title = e.get("title", "")
                        summary = e.get("summary", "")
                        text = f"{title}\n{summary}"

                        base_score, hits = score(text, watchlist)
                        w = source_weight(url)
                        weighted_score = min(int(base_score * w), 100)

                        if weighted_score >= threshold:
                            dt = entry_time(e)
                            rows.append({
                                "select": False,
                                "score": weighted_score,
                                "title": title,
                                "hits": ", ".join(hits),
                                "published": dt.isoformat(sep=" ", timespec="minutes") if dt else "",
                                "link": link,
                                "source": url,
                                "summary": summary,
                            })

                        if only_new:
                            seen.add(link)

            if only_new:
                save_seen(seen)

            if rows:
                df = pd.DataFrame(rows).sort_values(by=["score", "published"], ascending=[False, False])
                st.session_state["result_df"] = df
            else:
                st.session_state["result_df"] = None
                st.info("No matching items. Lower threshold or broaden your watchlist.")

        # reload df after fetch (important)
        df = st.session_state.get("result_df")
        if df is None or df.empty:
            st.write("Click **Fetch signals** to load news.")
            return

        # --- Filters (nice UX) ---
        with st.expander("Filters", expanded=True):
            min_score = st.slider("Min score", 0, 100, 30, 5)
            source_filter = st.multiselect("Source", sorted(df["source"].unique()))
            q = st.text_input("Search title/hits")

        view = df.copy()
        view = view[view["score"] >= min_score]
        if source_filter:
            view = view[view["source"].isin(source_filter)]
        if q:
            ql = q.lower()
            view = view[
                view["title"].str.lower().str.contains(ql, na=False)
                | view["hits"].str.lower().str.contains(ql, na=False)
            ]
        # --- Executive Summary button (paste here) ---
        st.divider()
        if st.button("🧾 Generate Executive Summary (Top 10)"):
            st.session_state["exec_summary_md"] = build_exec_summary(view)

        if "exec_summary_md" in st.session_state:
            st.markdown(st.session_state["exec_summary_md"])

        st.caption("Select rows to archive them.")

        edited_df = st.data_editor(
            view,
            column_config={
                "select": st.column_config.CheckboxColumn("Archive", help="Check to save", default=False),
                "link": st.column_config.LinkColumn("Link", display_text="Open"),
                "score": st.column_config.NumberColumn("Score", format="%d"),
            },
            disabled=["score", "title", "hits", "published", "link", "source"],
            hide_index=True,
            use_container_width=True,
            key="feed_editor",
        )

        # Auto-save selected
        selected_rows = edited_df[edited_df["select"] == True]
        if not selected_rows.empty:
            saved_items = load_saved()
            existing_links = {x.get("link") for x in saved_items}

            new_saves = 0
            for _, row in selected_rows.iterrows():
                if row["link"] not in existing_links:
                    saved_items.append({
                        "saved_at": datetime.now().isoformat(timespec="seconds"),
                        "title": row["title"],
                        "link": row["link"],
                        "hits": row["hits"],
                        "published": row["published"],
                        "score": int(row["score"]),
                        "source": row["source"],
                    })
                    existing_links.add(row["link"])
                    new_saves += 1

            if new_saves:
                save_saved(saved_items)
                st.toast(f"✅ Archived {new_saves} items!")

    # ---------------- TAB 2: SAVED ----------------
    with tab_saved:
        saved_data = load_saved()
        if not saved_data:
            st.info("Archive is empty. Save items from the Feed tab.")
            return

        saved_df = pd.DataFrame(saved_data).sort_values("saved_at", ascending=False)
        st.write(f"**Total Saved:** {len(saved_df)}")

        st.dataframe(
            saved_df,
            column_config={"link": st.column_config.LinkColumn("Link", display_text="Open")},
            use_container_width=True,
            hide_index=True
        )

        if st.button("🗑️ Clear Archive"):
            if os.path.exists(SAVE_FILE):
                os.remove(SAVE_FILE)
            # Clear the feed editor state so checked items don't immediately re-save
            if "feed_editor" in st.session_state:
                del st.session_state["feed_editor"]
            st.rerun()


run_app()
