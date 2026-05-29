# Day 40 — LLD: Task Management System (Trello-like)

**Topics:** Boards · Lists · Cards · Permissions · Activity Log

---

# 1. Problem Statement

Design a Trello-like task board: users create boards with lists and cards, drag cards between lists, assign members, and track activity.

---

# 2. Functional Requirements

| Feature | Description |
|---------|-------------|
| Board | Container with lists |
| List | Column (To Do, In Progress, Done) |
| Card | Task with title, description, due date, labels |
| Move Card | Reorder within/between lists |
| Members | Invite users to board with roles |
| Comments | Thread on card |
| Activity Log | Audit trail of changes |

---

# 3. Core Classes

```text
User
Board
  ├── id, name, ownerId, members, lists (ordered)
BoardMember
  ├── userId, role (OWNER, ADMIN, MEMBER, OBSERVER)

ListColumn
  ├── id, boardId, title, position

Card
  ├── id, listId, title, description, position, assignees, labels, dueDate

Comment
Activity
  ├── type, actorId, entityId, timestamp, payload

BoardService
  ├── createBoard(), addList(), createCard(), moveCard()
```

---

# 4. Move Card Logic

```java
@Transactional
public void moveCard(Long cardId, Long targetListId, int newPosition) {
    Card card = cardRepo.findById(cardId).orElseThrow();
    Long sourceListId = card.getListId();

    if (!sourceListId.equals(targetListId)) {
        cardRepo.decrementPositionsAfter(sourceListId, card.getPosition());
        card.setListId(targetListId);
    }
    cardRepo.incrementPositionsFrom(targetListId, newPosition);
    card.setPosition(newPosition);
    cardRepo.save(card);

    activityLog.record(ActivityType.CARD_MOVED, card);
}
```

Use fractional indexing (`position = 1024.5`) to reduce mass renumbering at scale.

---

# 5. Permissions

```java
public void assertCanEdit(Long userId, Long boardId) {
    BoardMember m = memberRepo.find(boardId, userId)
        .orElseThrow(() -> new ForbiddenException());
    if (m.getRole() == Role.OBSERVER) throw new ForbiddenException();
}
```

---

# 6. Class Diagram (Text)

```text
User 1 ---- * BoardMember * ---- 1 Board
Board 1 ---- * ListColumn 1 ---- * Card
Card 1 ---- * Comment
User 1 ---- * Activity
```

---

# 7. Real-time Collaboration

```text
moveCard() → publish CardMovedEvent → WebSocket to board subscribers
```

Optimistic UI on client; server authoritative on conflict.

---

# 8. Design Patterns

| Pattern | Use |
|---------|-----|
| Observer | Activity log + notifications |
| Command | Undo move (store inverse command) |
| Strategy | Notification channels (email, push) |
| Factory | Activity from event type |

---

# 9. Scale Considerations

- Partition by `boardId`
- Card positions as `double` or linked list (prev/next pointers)
- Full-text search on cards via Elasticsearch
- Archive old boards to cold storage

---

# 10. API Sketch

```java
POST   /boards
POST   /boards/{id}/lists
POST   /lists/{id}/cards
PATCH  /cards/{id}/move   { targetListId, position }
POST   /cards/{id}/comments
GET    /boards/{id}/activity
```

---

# Interview Questions

## Q1. How handle concurrent card moves?
Version field on card; last-write-wins or operational transform for position.

## Q2. Trello vs Jira LLD difference?
Jira adds workflows, sprints, epics — more state machine complexity.

## Q3. Soft delete?
`deletedAt` on card; list hides; restore within 30 days.

## Q4. Labels and custom fields?
`Label` entity many-to-many; `CustomField` EAV or JSON column on card.

## Q5. Notification on @mention?
Parse comment text; `NotificationService` fan-out to mentioned users.

---

# Quick Revision

```text
Trello LLD → Board > ListColumn > Card with position; moveCard reorders; roles + activity log; WebSocket for live updates.
```

---

*End of Day 40 LLD*
