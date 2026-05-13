from machine import Pin, ADC, PWM
import time

# --- ΡΥΘΜΙΣΕΙΣ ΜΟΤΕΡ ---
left_pwm = PWM(Pin(8))
left_dir = Pin(9, Pin.OUT)
right_pwm = PWM(Pin(10))
right_dir = Pin(11, Pin.OUT)

left_pwm.freq(1000)
right_pwm.freq(1000)

# --- ΡΥΘΜΙΣΕΙΣ ΑΙΣΘΗΤΗΡΩΝ ---
ir1 = Pin(0, Pin.IN)
ir2 = ADC(Pin(26))
ir3 = ADC(Pin(27))
ir4 = ADC(Pin(28))
ir5 = Pin(1, Pin.IN)

# --- ΠΑΡΑΜΕΤΡΟΙ ---
base_speed = 57000
KP = 32
KD = 23
THRESHOLD = 40000
last_error = 0

def drive(l_speed, r_speed):
    l_speed = max(0, min(65535, int(l_speed)))
    r_speed = max(0, min(65535, int(r_speed)))
    left_dir.value(0)
    right_dir.value(0)
    left_pwm.duty_u16(l_speed)
    right_pwm.duty_u16(r_speed)

def stop_motors():
    """Σταματάει τελείως τα μοτέρ"""
    left_pwm.duty_u16(0)
    right_pwm.duty_u16(0)
    print("Motors Stopped Safely.")

def read_sensors():
    s1 = 1 if ir1.value() == 1 else 0
    s2 = 1 if ir2.read_u16() > THRESHOLD else 0
    s3 = 1 if ir3.read_u16() > THRESHOLD else 0
    s4 = 1 if ir4.read_u16() > THRESHOLD else 0
    s5 = 1 if ir5.value() == 1 else 0
    return [s1, s2, s3, s4, s5]

# --- ΚΥΡΙΟ LOOP ΜΕ ΠΡΟΣΤΑΣΙΑ ---
try:
    print("Running... Press Stop in Thonny to halt.")
    while True:
        vals = read_sensors()
        active_sensors = sum(vals)
        
        if active_sensors > 0:
            weights = [-2, -1, 0, 1, 2]
            error = sum(vals[i] * weights[i] for i in range(5)) / active_sensors
            correction = (error * KP + (error - last_error) * KD) * 6000
            drive(base_speed + correction, base_speed - correction)
            last_error = error
        else:
            drive(0, 0) # Σταμάτα αν χάσεις τη γραμμή
            
        time.sleep(0.01)

except KeyboardInterrupt:
    # Αυτό τρέχει όταν πατάς το STOP στο Thonny (Ctrl+C)
    stop_motors()

finally:
    # Αυτό τρέχει ΠΑΝΤΑ πριν κλείσει το πρόγραμμα
    stop_motors()
    
    
    


