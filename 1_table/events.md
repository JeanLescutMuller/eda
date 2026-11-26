# Event logs: one row per click, call, request or event

Event tables are rarely analysed row by row: the questions are about users, sessions or days. Check the events, then build the level the question needs. Read `time.md` as well.

## The cells, in order

1. **The grain**: one row = one event of which kind, by whom, when. The event types and their counts (`px.histogram(y="event", text_auto=True)`). An event type with almost no rows, or one that appears only after a date, is orange (new tracking, a renamed event).
2. **Duplicate events**: the same user, event and timestamp (to the second or millisecond) more than once: a retry or double logging. Share > 1 % orange; per client or app version, red if concentrated in one.
3. **Events per user**: `df.groupby("user_id").size()`, `px.histogram(log_y=True)`. A long tail is normal; a few users with 100× the median are bots, test accounts or scrapers: show their top rows.
4. **Bot and test traffic**: users or ids matching test patterns (`test`, `qa`, `@example.com`, internal ids), user agents with `bot` / `crawler` / `headless`, events faster than a human can (two clicks under 100 ms), activity 24 hours a day. Their share of users and of events: orange; propose excluding them, and let the human decide.
5. **Ordering**: within a user, events that must follow an order (signup before purchase, open before close): the rows that break it. Red if any: clock skew, time zones, or out-of-order ingestion.
6. **Sessions**: a new session after a gap of inactivity (30 minutes is the common default; say the rule):

   ```python
   df_events = df_events.sort_values(["user_id", "ts"])
   gap = df_events.groupby("user_id").ts.diff() > pd.Timedelta("30min")
   df_events["session_id"] = gap.groupby(df_events.user_id).cumsum()
   ```

   Then sessions per user, events per session, session length (`px.histogram`). Sessions of hours or thousands of events: orange (bots, a tab left open).
7. **Missing identities**: events with no user id (anonymous, before login), and users with several ids (a login merging a device id). The share of events without a user: orange if > 10 %, since every per-user metric leaves them out.
8. **From events to the level of the question**: one table per user (or per user × day, per session) with the counts and firsts the analysis needs:

   ```python
   df_users = df_events.groupby("user_id").agg(
       first_ts=("ts", "min"),
       last_ts=("ts", "max"),
       n_events=("event", "size"),
       n_sessions=("session_id", "nunique"),
       purchased=("event", lambda s: (s == "purchase").any()),
   )
   ```

   Its row count against the unique users of the events table (green if equal). Pickle it: `df_users.pkl`.

Add each finding to the flags table of `table.md`.
