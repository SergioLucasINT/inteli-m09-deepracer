# inteli-m09-deepracer


Função de Recompensa: 

```

import math

def sigmoid(x):
    return 1 / (1 + math.exp(-x))

def waypoint_adjustment(waypoints, closest_waypoints, heading):
    next_point = waypoints[closest_waypoints[1]]
    prev_point = waypoints[closest_waypoints[0]]

    track_direction = math.atan2(next_point[1] - prev_point[1], next_point[0] - prev_point[0])
    track_direction = math.degrees(track_direction)

    direction_diff = abs(track_direction - heading)
    if direction_diff > 180:
        direction_diff = 360 - direction_diff

    DIRECTION_THRESHOLD = 10.0
    return sigmoid((DIRECTION_THRESHOLD - direction_diff) / 2)

def steering_angle(abs_steering):
    ABS_STEERING_THRESHOLD = 20.0
    return sigmoid((ABS_STEERING_THRESHOLD - abs_steering) / 5)

def speed(all_wheels_on_track, waypoints, closest_waypoints, current_speed):
    MAX_SPEED = 3.0
    MIN_SPEED = 0.5

    if not all_wheels_on_track:
        return 1e-3
    else:
        future_waypoints = waypoints[closest_waypoints[1]:closest_waypoints[1]+4]
        is_curve_ahead = False
        for i in range(len(future_waypoints) - 1):
            direction1 = math.atan2(future_waypoints[i+1][1] - future_waypoints[i][1],
                                    future_waypoints[i+1][0] - future_waypoints[i][0])
            direction2 = math.atan2(future_waypoints[i][1] - future_waypoints[i-1][1],
                                    future_waypoints[i][0] - future_waypoints[i-1][0])
            angle_diff = abs(math.degrees(direction1 - direction2))
            if angle_diff > 10: 
                is_curve_ahead = True
                break

        if is_curve_ahead:
            return max(MIN_SPEED, current_speed - 0.5)
        else:
            return min(MAX_SPEED, current_speed + 0.5)

def reward_function(params):
    waypoint_reward = waypoint_adjustment(params['waypoints'], params['closest_waypoints'], params['heading'])
    steering_reward = steering_angle(abs(params['steering_angle']))
    speed_reward = speed(params['all_wheels_on_track'], params['waypoints'], params['closest_waypoints'], params['speed'])

    alignment_steering_factor = waypoint_reward * steering_reward
    speed_factor = math.exp(speed_reward - 1) 

    if alignment_steering_factor < 0.5:
        speed_factor *= 0.5 

    return alignment_steering_factor * speed_factor


```
