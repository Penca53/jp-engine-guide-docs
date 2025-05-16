# Transition

## From Source to Destination

A Transition within the FSM (Finite State Machine) is defined by a source state and a destination state. Additionally, it can include an optional boolean condition. The FSM evaluates this condition to determine if the transition is valid and can be taken. This condition function is restricted to using only the provided `TContext` to access any data required for its true/false evaluation; no other external data sources are permitted.

{% code title="engine/transition.h" %}
```cpp
#pragma once

#include <functional>
#include <string>

#include "state.h"

namespace ng {

/// @brief Represents a transition between two states in a state machine, triggered by a specific condition.
/// @tparam TContext The type of the context object that the state machine operates on.
template <typename TContext>
class Transition {
 public:
  /// @brief Constructs a Transition with a source state, a target state, and a condition function.
  /// @param from The ID of the state to transition from.
  /// @param to The ID of the state to transition to.
  /// @param condition A function that takes the context object and returns true if the transition condition is met, false otherwise.
  Transition(State<TContext>::ID from, State<TContext>::ID to,  // NOLINT
             std::function<bool(TContext&)> condition)
      : from_(std::move(from)),
        to_(std::move(to)),
        condition_(std::move(condition)) {}

  /// @brief Returns the ID of the state this transition originates from.
  /// @return The ID of the source state.
  [[nodiscard]] const State<TContext>::ID& GetFrom() const { return from_; }

  /// @brief Returns the ID of the state this transition leads to.
  /// @return The ID of the target state.
  [[nodiscard]] const State<TContext>::ID& GetTo() const { return to_; }

  /// @brief Evaluates the transition condition with the given context.
  /// @param context A reference to the context object.
  /// @return True if the condition for this transition is met, false otherwise.
  [[nodiscard]] bool MeetsCondition(TContext& context) const {
    return condition_(context);
  }

 private:
  // The ID of the state to transition from.
  State<TContext>::ID from_;
  // The ID of the state to transition to.
  State<TContext>::ID to_;
  // The function that determines if the transition should occur.
  std::function<bool(TContext&)> condition_;
};

}  // namespace ng
```
{% endcode %}
