# FSM

## Finite State Machine

An FSM (Finite State Machine) is a computational model that represents a system with a finite number of possible states and transitions between those states. The system can only be in one state at any given time. Its behavior is determined by its current state and the transitions that occur based on specific conditions or inputs.

{% code title="engine/fsm.h" %}
```cpp
#pragma once

#include <memory>
#include <string>
#include <unordered_map>
#include <vector>

#include "state.h"
#include "transition.h"

namespace ng {

/// @brief Implements a finite state machine (FSM) to manage the behavior of an object based on its current state and transitions.
/// @tparam TContext The type of the context object that the state machine operates on.
template <typename TContext>
class FSM {
 public:
  /// @brief Constructs an FSM with a context object and an initial entry state.
  /// @param context A pointer to the context object that the FSM will operate on. This pointer must not be null.
  /// @param entry_state A unique pointer to the initial state of the FSM. Ownership is transferred to the FSM.
  FSM(TContext* context, std::unique_ptr<State<TContext>> entry_state)
      : context_(context), current_state_(entry_state.get()) {
    assert(context);
    AddState(std::move(entry_state));
  }

  /// @brief Returns the context object associated with this FSM.
  /// @return A pointer to the context object. Never null.
  TContext* GetContext() { return context_; }

  /// @brief Adds a new state to the FSM.
  /// @param state A unique pointer to the State object to add. Ownership is transferred to the FSM.
  void AddState(std::unique_ptr<State<TContext>> state) {
    state->SetContext(context_);
    states_.insert({state->GetID(), std::move(state)});
  }

  /// @brief Adds a new transition to the FSM.
  /// @param transition The Transition object to add. It defines the source state, target state, and the condition for the transition.
  void AddTransition(Transition<TContext> transition) {
    auto it = transitions_.find(transition.GetFrom());
    if (it == transitions_.end()) {
      it = transitions_.insert({transition.GetFrom(), {}}).first;
    }

    it->second.push_back(std::move(transition));
  }

  /// @brief Updates the FSM. Checks for applicable transitions from the current state and updates the current state.
  void Update() {
    auto it = transitions_.find(current_state_->GetID());
    if (it != transitions_.end()) {
      for (auto& transition : it->second) {
        if (transition.MeetsCondition(*context_)) {
          Transit(states_.at(transition.GetTo()).get());
          break;
        }
      }
    }

    current_state_->Update();
  }

 private:
  /// @brief Performs a transition from the current state to a new state.
  /// @param to_state A pointer to the State object to transition to. This pointer must not be null.
  void Transit(State<TContext>* to_state) {
    assert(to_state);
    current_state_->OnExit();
    current_state_ = to_state;
    current_state_->OnEnter();
  }

  // Pointer to the context object that the FSM operates on. Never null after construction.
  TContext* context_ = nullptr;

  // Stores all the states of the FSM, indexed by their unique ID.
  std::unordered_map<std::string, std::unique_ptr<State<TContext>>> states_;
  // Stores the transitions of the FSM, where the key is the source state ID and the value is a vector of transitions.
  std::unordered_map<std::string, std::vector<Transition<TContext>>>
      transitions_;
  // Pointer to the currently active state. Never null after construction.
  State<TContext>* current_state_ = nullptr;
};

}  // namespace ng
```
{% endcode %}
