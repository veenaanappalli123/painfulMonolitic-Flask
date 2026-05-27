# GameHub — Understanding the system

10 questions to test your understanding of the data flow and architecture.
Work through them in order: read the code first, then run the app, then try to break things.

---

## How to investigate

You will need three things:

**1. Read the source code**
Start with `models.py` (the schema), then `seed.py` (the data), then `app.py` (the logic).
Many questions are answered entirely by reading carefully.

**2. Run the app and interact with it**
Use the UI at `http://localhost:5000` or send requests with curl or Postman.
Observe what actually happens — don't just reason about it.

```bash
# Example: log an activity for nova (id=1) on Hollow Knight (id=1)
curl -X POST http://localhost:5000/activities \
  -H "Content-Type: application/json" \
  -d '{"user_id": 1, "game_id": 1, "action": "started"}'
```

**3. Query the database directly**
Open `gamehub.db` with a SQLite tool and inspect the actual rows.

```bash
sqlite3 gamehub.db
.tables
SELECT COUNT(*) FROM notifications;
SELECT * FROM notifications WHERE user_id = 1;
```

Or use a GUI: **DB Browser for SQLite** (free, recommended).

---

## Suggested approach

| Phase | Questions | What you are doing |
|-------|-----------|-------------------|
| Read first | 1, 4, 8, 10 | Understand the code before touching anything |
| Then run it | 3, 6, 9    | Observe actual behaviour                     |
| Then break it | 2, 5, 7  | Try things, hit walls, reason about why      |

---

## Questions

**1.** When a user logs a new activity, how many database tables are written to?
List them and explain why each one is affected.

**answer=activities:because tehre is a new activity with their id, game id,
notifiactions:since we can see it is implemented , to ur friedns they are notified, so a chnage to the table as well,I first checked the create_activity() route in app.py, then I tested it by checking the database before and after logging an activity.
I noticed that logging one activity does not only affect the activities table,,
The app first inserts the activity into the activities table. Then it fetches the user’s friends and creates notification rows for them in the notifications table.


---

**2.** You call `DELETE FROM users WHERE id = 3` directly in SQLite.
What happens, and why? What would you need to do instead?
 it throws a foreign key constraint error  because other tables still referrence that user.
 i would need to manually delete dependent rows first, cus the schema enables foreign keys but no cascade deletion is there,so sql will block the deltions

---

**3.** User `nova` changes her username to `nova_2`.
She then checks her friends' notification feeds.
What do they see — the old name or the new one? Why?
old name:because the notification messages are stored as texts , normal plain texts , so no dynamic change,for the old notifications..::::...for the new ones , it will show nova_2, if the notification was created after the change

---

**4.** Trace the full journey of a `POST /activities` request.
Starting from the HTTP call, list every operation that happens before the response is returned.

---

**5.** `pixel_queen` opts out of activity tracking.
A teammate adds an `opted_out` boolean column to the `users` table and updates the `POST /activities` API route to check it.
Is the feature fully implemented? What did they miss?
no
At first I thought it was enough to update the POST /activities route,: but after searching through the code ,,, there are actually two activity creation routes,,
POST /activities ---JSON API route
POST /view/activities ------HTML form route
If only the API route checks opted_out, users can still create activities through the web UI.


---

**6.** How many rows are created in the database when `nova` logs one activity, given the current seed data?
Show your working.
tried creating an activity using nova, he is friends with 3 people, they all got notified , the databse rows were created 
4 created 
1 row is inserted into activities
3 rows are inserted into notifications:he got 3 friend so 3 notoficationsss

---

**7.** You need to delete `maya_r`.
In what order must you delete rows across the tables, and why does the order matter?


---

**8.** The `notifications` table has a foreign key pointing to `activities`.
What happens if you try to delete an activity that has notifications attached to it?

error:cus the notification table is connected from activities, so when u delte an activity the notifucations will point to what?????:sql threw a foreign key constrain error, they r connected

---

**9.** A bug is found in the game catalog — wrong genre for one game.
You fix it and restart the app to ship the change.
What else just went down, and for how long?


it will take down the ,users,activities,notifications,friendships
the web UI
the API
just a catalog changed but that would take down the compenets of this app ,cuz they are deployed togetehr as one ,the entire application uses one flask application .


---

**10.** A teammate says: *"let's just move the notification logic into its own function in `app.py`"*.
Does that solve the problem described in Task 4?
What is the actual architectural issue?

separate notifications into their own service so activities ,notifications could process them independently, now the architecture forces it to run in the same flask app, and share the same databaseee,dependency too much, they are tight in this restriction and it raises error if u r not careful
