# Day 38 — LLD: Meeting Scheduler (Calendar)

**Topics:** Requirements · Interval Overlap · Room Allocation · Design

---

# 1. Problem Statement

Design a meeting scheduler where users book meetings in rooms with start/end times, supporting conflict detection and optional recurring meetings.

---

# 2. Functional Requirements

| Feature | Description |
|---------|-------------|
| Book Meeting | User, room, start, end, attendees |
| Cancel Meeting | Remove booking |
| List Availability | Free slots for room or user |
| Conflict Check | No double-book room or required attendee |
| Recurring | Weekly standup pattern |

---

# 3. Core Classes

```text
User
Meeting
  ├── id, organizer, attendees, roomId, start, end, status
Room
  ├── id, capacity, building, equipment

CalendarService
  ├── bookMeeting(), cancel(), getAvailability()
ConflictChecker
  ├── hasConflict(room, interval)
  ├── hasAttendeeConflict(user, interval)
```

---

# 4. Interval Overlap Logic

Two intervals `[s1,e1)` and `[s2,e2)` overlap iff:

```text
s1 < e2 AND s2 < e1
```

```java
public boolean overlaps(Instant start1, Instant end1, Instant start2, Instant end2) {
    return start1.isBefore(end2) && start2.isBefore(end1);
}
```

---

# 5. Book Meeting — Sequence

```text
User -> CalendarController.book(request)
     -> CalendarService.book()
         -> validate end > start
         -> ConflictChecker for room
         -> ConflictChecker for each required attendee
         -> persist Meeting
         -> send invites (email/notification)
     -> return MeetingResponse
```

---

# 6. Find Available Slots

```java
public List<TimeSlot> getAvailability(Long roomId, LocalDate date) {
    List<Meeting> meetings = repo.findByRoomAndDate(roomId, date);
    meetings.sort(Comparator.comparing(Meeting::getStart));
    List<TimeSlot> free = new ArrayList<>();
    Instant dayStart = date.atStartOfDay(ZoneId.systemDefault()).toInstant();
    Instant dayEnd = date.plusDays(1).atStartOfDay(ZoneId.systemDefault()).toInstant();
    Instant cursor = dayStart;
    for (Meeting m : meetings) {
        if (cursor.isBefore(m.getStart())) {
            free.add(new TimeSlot(cursor, m.getStart()));
        }
        cursor = m.getEnd().isAfter(cursor) ? m.getEnd() : cursor;
    }
    if (cursor.isBefore(dayEnd)) free.add(new TimeSlot(cursor, dayEnd));
    return free;
}
```

---

# 7. Recurring Meetings

```java
public class RecurrenceRule {
    private RecurrenceType type; // DAILY, WEEKLY, MONTHLY
    private DayOfWeek dayOfWeek;
    private LocalDate until;
}
```

Expand occurrences on read or pre-materialize N instances — trade storage vs query complexity.

---

# 8. Concurrency

```java
@Transactional
public Meeting book(BookRequest req) {
    lockRoom(req.getRoomId()); // SELECT FOR UPDATE on room row
    if (conflictChecker.hasConflict(...)) throw new ConflictException();
    return repo.save(toMeeting(req));
}
```

---

# 9. Design Patterns

| Pattern | Use |
|---------|-----|
| Strategy | Different conflict policies (allow optional attendees overlap) |
| Factory | Recurrence expansion |
| Observer | Notify attendees on book/cancel |

---

# 10. Scale Considerations

- Index `(room_id, start, end)` for range queries
- Shard by `orgId` for enterprise
- Cache today's meetings per room in Redis

---

# Interview Questions

## Q1. Google Calendar scale?
Event store per user; fan-out invites; eventual consistency for free-busy across orgs.

## Q2. Timezone handling?
Store UTC in DB; convert at API boundary using `ZonedDateTime` / user's `ZoneId`.

## Q3. DST edge cases?
Use `java.time` — never raw `Calendar`; test spring-forward/fall-back days.

## Q4. Find meeting room for N people?
Filter rooms by capacity, then run availability scan.

## Q5. Optimistic vs pessimistic booking?
Optimistic: insert with unique constraint on (room, start). Pessimistic: row lock during book.

---

# Quick Revision

```text
Meeting Scheduler → overlap check s1<e2 && s2<e1; lock room; expand recurrence; UTC storage.
```

---

*End of Day 38 LLD*
