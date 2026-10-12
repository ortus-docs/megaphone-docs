# DatabaseNotificationKeysetCursor

getNotificationsAfter returns this forward-only cursor. getResults() returns authorized DatabaseNotification objects. getNextCursor() returns the opaque next token; hasNext() reports whether another page exists. next() advances this cursor and returns it, preserving filters and authorization constraints. Check hasNext before advancing. getMemento() returns results, nextCursor, and hasMore for serialization. Existing numbered-page cursors remain available.
