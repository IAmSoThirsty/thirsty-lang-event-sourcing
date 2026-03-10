<div align="right">
  <img src="https://img.shields.io/badge/Date-2026--03--10-blue?style=for-the-badge" alt="Date" />
  <img src="https://img.shields.io/badge/Status-Active-success?style=for-the-badge" alt="Status" />
  <img src="https://img.shields.io/badge/Tier-Master-gold?style=for-the-badge" alt="Tier" />
</div>

# Thirsty-lang Event Sourcing 💧📜

Event-sourced architecture with CQRS, event store, and projections.

## Features

- Event store (append-only log)
- CQRS pattern
- Event replaying
- Projections & read models
- Snapshots for performance
- Event versioning

## Event Store

```thirsty
glass EventStore {
  drink events
  
  glass append(streamId, event) {
    shield eventProtection {
      sanitize event
      
      events.push(reservoir {
        streamId: streamId,
        eventType: event.type,
        data: event.data,
        version: getNextVersion(streamId),
        timestamp: Date.now()
      })
    }
  }
  
  glass getEvents(streamId) {
    return events.filter(e => e.streamId == streamId)
  }
  
  glass replay(streamId, aggregate) {
    drink events = getEvents(streamId)
    
    refill drink event in events {
      aggregate.apply(event)
    }
    
    return aggregate
  }
}
```

## CQRS Pattern

```thirsty
// Command
glass CreateUserCommand {
  drink userId
  drink email
  drink name
}

// Command Handler
glass UserCommandHandler {
  glass handle(command) {
    shield commandProtection {
      sanitize command
      
      thirsty command instanceof CreateUserCommand
        drink event = UserCreatedEvent(command)
        eventStore.append(`user-${command.userId}`, event)
    }
  }
}

// Event
glass UserCreatedEvent {
  drink type = "UserCreated"
  drink data
  
  glass constructor(command) {
    data = reservoir {
      userId: command.userId,
      email: command.email,
      name: command.name
    }
  }
}

// Projection
glass UserProjection {
  glass project(event) {
    thirsty event.type == "UserCreated"
      await db.Users.insert(event.data)
  }
}
```

## License

MIT
