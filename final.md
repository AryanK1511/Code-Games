```
# ===== LAST TICK VARIABLES SAVER =====
function save_vars()
  last_enemy_visible = enemy_visible
  last_enemy_angle = enemy_angle
  last_enemy_distance = enemy_distance
  last_energy = energy
  last_speed = speed
  last_direction = direction
end

# ===== HELPER FUNCTIONS =====

# --- Zig Zag Motion ----
function zig_zag_motion()
  # Set a random turn amount if it is 0
  if turn_amount == 0
    turn_amount = rand(5, 15)
  end

  # Switch the direction of turning if the turning variable is exhausted
  if turning <= 0
    turning = rand(20, 50)
    turn_amount *= -1
  end

  # Turn in some direction and reduce the turning variable
  if turning > 0
    right(turn_amount)
    turning -= 1
  end
end

# --- Follow spaceships and fire ---
function follow_and_fire()
  # Close the distance
  if enemy_distance > 300
    rift
  end
  
  # Constants
  shot_speed = 15 # speed of the shot
  
  # Calculate the delta values for angle and distance
  delta_angle = enemy_angle - last_enemy_angle
  delta_distance = enemy_distance - last_enemy_distance
  delta_speed = speed - last_speed

  # Make sure delta_angle is normalized between -90 and +90
  if delta_angle > 90
    delta_angle -= 180
  else if delta_angle < -90
    delta_angle += 180
  end

  # Estimate the enemy's current speed and direction
  enemy_speed = sqrt(delta_distance * delta_distance + last_enemy_distance * last_enemy_distance - 2 * delta_distance * last_enemy_distance * cos(delta_angle))
  enemy_direction_change = atan2(delta_distance * sin(delta_angle), last_enemy_distance + delta_distance * cos(delta_angle))

  # Calculate the time it takes for a projectile to reach the enemy's current position
  time_to_hit = enemy_distance / shot_speed

  # Predict the enemy's future position considering their speed and direction change
  predicted_enemy_angle = enemy_angle + delta_angle * time_to_hit + enemy_direction_change * time_to_hit
  predicted_enemy_distance = enemy_distance + enemy_speed * time_to_hit

  # Normalize predicted_enemy_angle to be between -180 and +180 for turning decisions
  if predicted_enemy_angle > 180
    predicted_enemy_angle -= 360
  else if predicted_enemy_angle < -180
    predicted_enemy_angle += 360
  end

  # Determine the direction to turn based on the predicted angle
  if predicted_enemy_angle > 0
    right(predicted_enemy_angle)  # Enemy is predicted to be on the right
  else
    left(-predicted_enemy_angle)  # Enemy is predicted to be on the left
  end

  # Adjust speed to maintain optimal distance
  faster(enemy_distance - 130)

  # Fire if the enemy is within a reasonable distance and visible
  if enemy_distance <= 250
    fire
  end
end

# --- Long range evasion ---
function turn_around_and_seek()
  slower(5)
  right(360)
end

# --- Close range evasion ---
function rift_and_seek()
  # Go faster and rift if possible for wrap around
  faster(5)
  rift
end

# --- The default evasion strategy ---
function evade_and_seek()
  if speed <= 3
    faster(1)
  end
  zig_zag_motion
end

function manage_energy_with_ticks()
  # Update current energy with the actual energy value from the game
  current_energy = energy

  # Increment the ticks counter
  ticks_counter += 1

  # Every 50 ticks, update energy_last_second
  if ticks_counter == 50
    energy_last_second = current_energy
    ticks_counter = 0  # Reset the counter for the next cycle
  end
end

# --- Rift evasion in case of emergency ---
function activate_emergency_evac()
  if enemy_distance < 200
    activated = false
    # Check if we got hit and reset the ticks_to_next_hit counter
    if energy < energy_last_second
      hello += 1
      activated = true
      ticks_to_next_hit = 50 # Assume the enemy will shoot again as soon as they can
    end

    # Decrement the tick counter only if it's greater than 0
    if ticks_to_next_hit > 0
      ticks_to_next_hit -= 1
    end

    # Start evasive maneuver if the enemy is visible and the counter is low
    if activated == true
      # Is 10 ticks enough to escape? If not, consider adjusting this value
      rift_and_turn
      bye += 1
      activated = false
    end
  end
end

# --- Rift and Turn Function ---
function rift_and_turn()
  faster(5)
  rift # Engage a rapid movement change to evade incoming fire
  turn_amount = 360 # Choose a random angle to turn, avoiding predictability
  if rand(0, 1) == 0
    left(turn_amount)
  else
    right(turn_amount)
  end
end

# =========================
# ===== MAIN FUNCTION =====
# =========================
function play_game()
  manage_energy_with_ticks
  # Evade when emergency 
  activate_emergency_evac
  # If enemy is not visible then try to locate 
  if !enemy_visible
    # We just lost sight of the enemy
    if last_enemy_visible
      # If the enemy was far
      if last_enemy_distance > 200 
        turn_around_and_seek
      # We are going fast and enemy is close, so its better to wrap around
      else if last_enemy_distance < 20 and speed > 4
        rift_and_seek
      end  
    # Choose default strategy
    else
      evade_and_seek
    end 
  # If enemy is visible, then attack
  else
    follow_and_fire
  end
   
  # Update vars to save previous state
	save_vars    
end

# Calling the play_game() function and executing the logic
play_game
```