Web VPython 3.2
from vpython import *

### Default Settings ###
scene = canvas(width=550, height=550, background=color.black, align='left')
local_light(pos=vector(0, 0, 1000), color=color.white)

### Move View ###
def move_view(evt):
    key = evt.key
    shift_amount = 100
    
    if key == 'left':
        scene.center.x -= shift_amount
    elif key == 'right':
        scene.center.x += shift_amount
    elif key == 'up':
        scene.center.y += shift_amount
    elif key == 'down':
        scene.center.y -= shift_amount
        
scene.bind('keydown', move_view)

### Constants ###
epsilon_0 = 8.854e-12
voltage = 10.0
dielectric_constant = 5.0
initial_gap = 10.0
initial_size = 20.0
plate_thickness = 30.0

### Objects ###
battery = box(pos=vector(-3000.0, 0, 0), size=vector(1000.0, 2000.0, 1000.0), color=vector(1.0, 0.702, 0.729), shininess = 1)
battery_label = text(text='Battery', pos = battery.pos + vector(0, 1500.0, 0), height = 200, color=color.white, align='center', font='serif', billboard=True)
battery_plus = text(text='+', pos=battery.pos+vector(0,battery.size.y/2+100,0), height=200, color=color.white, align='center', font='serif', billboard=True)
battery_minus = text(text='-', pos=battery.pos-vector(0,battery.size.y/2+200,0), height=200, color=color.white, align='center', font='serif', billboard=True)

capacitor_plate1 = box(pos=vector(2000.0, 100 * initial_gap / 2, 0), size=vector(initial_size, plate_thickness, 2000.0), color=vector(0.8588, 0.8627, 1.0), shininess = 1)
capacitor_plate2 = box(pos=vector(2000.0, -100 * initial_gap / 2, 0), size=vector(initial_size, plate_thickness, 2000.0), color=vector(0.8588, 0.8627, 1.0), shininess = 1)
capacitor_text = text(text='Capacitor', pos = capacitor_plate1.pos + vector(0, 500, 0), height = 200, color=color.white, align='center', font='serif', billboard=True)

dielectric = box(pos=vector(2000, 0, 0), size=vector(initial_size, initial_gap * 10, 2000.0), color=vector(0.7725, 0.8784, 0.6549), opacity=0.75)
dielectric_text = text(text='Dielectric', pos = dielectric.pos + vector(dielectric.size.x+2000, 0, 0), height = 150, color = color.white, align='center', font='serif', billboard=True)

led = box(pos=vector(-3000.0, 0, 0), size=vector(1000.0, 2000.0, 1000.0), color=color.yellow, visible = False, opacity = 0.1)
inner_led = box(pos=led.pos, size=led.size-vector(500, 500, 500), color=color.white, opacity = 1, shininess=1, visible = False)
led_text = text(text='LED', pos = battery.pos + vector(0, 1500.0, 0), height = 200, color=color.white, align='center', visible = False, font='serif', billboard=True)

wire1 = cylinder(pos=battery.pos + vector(0, 1000.0, 0), axis=vector(0, 1500.0, 0), radius=20.0, color=vector(1,0.7,0.2), texture={'file':textures.metal})
wire2 = cylinder(pos=wire1.pos + wire1.axis, axis=vector(5000.0, 0, 0), radius=20.0, color=vector(1,0.7,0.2), texture={'file':textures.metal})
wire3 = cylinder(pos=wire2.pos + wire2.axis,
                 axis=vector(0, capacitor_plate1.pos.y - (wire1.pos.y + wire1.axis.y), 0), 
                 radius=20.0, color=vector(1,0.7,0.2), texture={'file':textures.metal})

wire4 = cylinder(pos=battery.pos + vector(0, -1000.0, 0), axis=vector(0, -1500.0, 0), radius=20.0, color=vector(1,0.7,0.2), texture={'file':textures.metal})
wire5 = cylinder(pos=wire4.pos + wire4.axis, axis=vector(5000.0, 0, 0), radius=20.0, color=vector(1,0.7,0.2), texture={'file':textures.metal})
wire6 = cylinder(pos=wire5.pos + wire5.axis, 
                 axis=vector(0, capacitor_plate2.pos.y - (wire4.pos.y + wire4.axis.y), 0), 
                 radius=20.0, color=vector(1,0.7,0.2), texture={'file':textures.metal})

### Functions ###
# Drag-and-drop
is_dragging = False
dragging = None

def start_drag(evt):
    global dragging
    obj = scene.mouse.pick
    if obj == dielectric:
        dragging = obj
        dragging.start_drag_pos = dragging.pos - scene.mouse.pos
        dielectric.opacity = 1.0
        update_dielectric_text()

def drag(evt):
    global dragging
    if dragging:
        dragging.pos = scene.mouse.pos + dragging.start_drag_pos
        update_dielectric_text()

def end_drag(evt):
    global dragging
    dielectric.opacity = 0.75
    if abs(dielectric.pos.x - 2000.0) < 500 and abs(dielectric.pos.z) < 1000:
        dielectric.pos = vector(2000.0, 0, 0)  # Center in the capacitor
        insert_dielectric()
        dielectric.opacity = 0.5
    else:
        remove_dielectric()
    dragging = None
    update_dielectric_text()

scene.bind('mousedown', start_drag)
scene.bind('mousemove', drag)
scene.bind('mouseup', end_drag)

# Lists to hold charge markers
positive_charges = []
negative_charges = []
charge_vectors = []
dielectric_inserted = False
battery_connected = True

# Initialize global variables
charge = 0
capacitance = 0

# Update charge markers
def update_charges(plate, charge_type, num_charges):
    charge_radius = 15

    if charge_type == 'positive':
        while len(positive_charges) < num_charges:
            charge_marker = sphere(radius=charge_radius, color=color.red, visible=False, texture = {'file': textures.metal}, shininess=1, emissive = True)
            positive_charges.append(charge_marker)
        while len(positive_charges) > num_charges:
            charge_marker = positive_charges.pop()
            charge_marker.visible = False

        grid_size = ceil(sqrt(num_charges)) if num_charges > 0 else 1
        num_rows = grid_size
        num_columns = ceil(num_charges / num_rows)

        x_spacing = plate.size.x / max(num_columns, 1)
        z_spacing = plate.size.z / max(num_rows, 1)

        for index, charge_marker in enumerate(positive_charges):
            i = index % num_columns
            j = index // num_columns
            x_pos = plate.pos.x + (i - (num_columns - 1) / 2) * x_spacing
            z_pos = plate.pos.z + (j - (num_rows - 1) / 2) * z_spacing
            charge_marker.pos = vector(x_pos, plate.pos.y - plate.size.y / 2 - charge_radius, z_pos)
            charge_marker.visible = True

    else:
        while len(negative_charges) < num_charges:
            charge_marker = sphere(radius=charge_radius, color=color.blue, visible=False, texture = {'file': textures.metal}, shininess=1)
            negative_charges.append(charge_marker)
        while len(negative_charges) > num_charges:
            charge_marker = negative_charges.pop()
            charge_marker.visible = False

        grid_size = ceil(sqrt(num_charges)) if num_charges > 0 else 1
        num_rows = grid_size
        num_columns = ceil(num_charges / num_rows)

        x_spacing = plate.size.x / max(num_columns, 1)
        z_spacing = plate.size.z / max(num_rows, 1)

        for index, charge_marker in enumerate(negative_charges):
            i = index % num_columns
            j = index // num_columns
            x_pos = plate.pos.x + (i - (num_columns - 1) / 2) * x_spacing
            z_pos = plate.pos.z + (j - (num_rows - 1) / 2) * z_spacing
            charge_marker.pos = vector(x_pos, plate.pos.y + plate.size.y / 2 + charge_radius, z_pos)
            charge_marker.visible = True

    update_vectors()

# Update vectors between corresponding charge markers
def update_vectors():
    while len(charge_vectors) > min(len(positive_charges), len(negative_charges)):
        vec = charge_vectors.pop()
        vec.visible = False

    while len(charge_vectors) < min(len(positive_charges), len(negative_charges)):
        vec = arrow(shaftwidth=15, visible=False)
        charge_vectors.append(vec)

    space_offset = 25

    for i in range(len(charge_vectors)):
        start_pos = positive_charges[i].pos
        end_pos = negative_charges[i].pos
        direction = norm(end_pos - start_pos)
        
        start_offset = start_pos + direction * (positive_charges[i].radius + space_offset)
        end_offset = end_pos - direction * (negative_charges[i].radius + space_offset)
        
        charge_vectors[i].pos = start_offset
        charge_vectors[i].axis = end_offset - start_offset
        charge_vectors[i].visible = True
        charge_vectors[i].color = color.white
        charge_vectors[i].shaftwidth = 15

def update_capacitor():
    global charge, capacitance, battery_connected  # Declare global variables
    gap = gap_slider.value
    size = area_slider.value
    dielectric_constant = dielectric_slider.value
    
    # Update capacitor plate positions and sizes
    capacitor_plate1.pos = vector(2000.0, 100 * gap / 2, 0)
    capacitor_plate2.pos = vector(2000.0, -100 * gap / 2, 0)
    
    capacitor_plate1.size = vector(size * 100, plate_thickness, 2000.0)
    capacitor_plate2.size = vector(size * 100, plate_thickness, 2000.0)
    
    # Update dielectric size and position to match the gap
    if dielectric.pos == vector(2000.0, 0, 0):
        dielectric.size = vector(size * 100, gap * 100, 2000.0)  # Set the size of the dielectric to match the gap
    
    # Update wire positions
    wire3.pos = wire2.pos + wire2.axis
    wire3.axis = vector(0, capacitor_plate1.pos.y - (wire1.pos.y + wire1.axis.y), 0)
    wire6.pos = wire5.pos + wire5.axis
    wire6.axis = vector(0, capacitor_plate2.pos.y - (wire4.pos.y + wire4.axis.y), 0)
    
    # Calculate capacitance and charge, accounting for dielectric if inserted
    dielectric_factor = dielectric_constant if dielectric.pos == vector(2000.0, 0, 0) else 1.0
    area = size * 2000.0 * 1e-6  # Plate area in m^2 (convert mm^2 to m^2)
    gap_m = gap * 1e-3  # Convert gap to meters

    capacitance = epsilon_0 * dielectric_factor * area / gap_m  # Capacitance in Farads

    if battery_connected:
        charge = capacitance * voltage  # Calculate charge based on capacitance and voltage
    
    # Update display texts
    capacitance_text.text = '        C (Capacitance): {:.2e} pF'.format(capacitance * 1e12)
    charge_text.text = '        Q (Charge): {:.2e} pC'.format(charge * 1e12)

    # Update slider text displays
    gap_slider_text.text = '        d (Distance between Plates): {:.1f} mm'.format(gap)
    area_slider_text.text = '        S (Area of Plates): {:.1f} mm\u00B2'.format(size)
    dielectric_slider_text.text = '        ε (Dielectric Constant): {:.1f}\n'.format(dielectric_constant)
     
    # Update charge markers
    num_charges = int(abs(charge) * 7 * 1e10)  # Scale factor for visible effect
    update_charges(capacitor_plate1, 'positive', num_charges)
    update_charges(capacitor_plate2, 'negative', num_charges)
    
# Connect LED
def connect_capacitor_to_led():
    global battery_connected, charge  # Declare global variables
    battery_connected = False  # Battery is disconnected
    battery.visible = False
    battery_label.visible = False
    battery_plus.visible = False
    battery_minus.visible = False
    led.visible = True
    inner_led.visible = True
    led_text.visible = True
    
    charge = float(charge_text.text.split(": ")[1].split(" pC")[0]) * 1e-12  # Convert from pC to C
    capacitance = float(capacitance_text.text.split(": ")[1].split(" pF")[0]) * 1e-12  # Convert from pF to F

    scene.background = color.white  # Start with a white background
    time_step = 0.00001  # Smaller time step for finer control
    while charge > 0:
        rate(250)  # Higher rate to update more frequently
        charge -= charge / (capacitance * 1e7) * time_step  # Reduced discharge rate for smoother transition
        charge = max(1e-12, charge)  # Ensure charge does not go below 1e-12C
        if charge == 1e-12:
            charge = 0
        brightness = (charge / (capacitance * voltage))**2  # Calculate brightness based on remaining charge
        gray_value = max(0, min(1, brightness))  # Clamp the gray value between 0 and 1
        
        scene.background = vector(gray_value, gray_value, gray_value)  # Gradually update the background color
        if gray_value >= 0.1:
            inner_led.color = vector(gray_value, gray_value, gray_value)
        else:
            inner_led.color = color.black  # Turn off the LED when brightness is too low
            
        # Update charges and display text
        update_charges(capacitor_plate1, 'positive', int(abs(charge) * 7 * 1e10))
        update_charges(capacitor_plate2, 'negative', int(abs(charge) * 7 * 1e10))
        charge_text.text = '        Q (Charge): {:.2e} pC'.format(charge * 1e12)

# Reconnect the battery
def reconnect_battery():
    global battery_connected, charge, capacitance  # Declare global variables
    battery_connected = True  # Battery is connected
    led.visible = False
    inner_led.visible = False
    led_text.visible = False
    battery.visible = True
    battery_label.visible = True
    battery_plus.visible = True
    battery_minus.visible = True
    scene.background = color.black
    
    # Reset charge based on the updated capacitance
    update_capacitor()  # This will recalculate charge because battery_connected is True

# Insert dielectric into the capacitor
def insert_dielectric():
    global dielectric_inserted
    dielectric_inserted = True    
    update_capacitor()

def remove_dielectric():
    global dielectric_inserted
    dielectric_inserted = False
    update_capacitor()

def update_dielectric_text():
    dielectric_text.pos = dielectric.pos + vector(dielectric.size.x+80, 0, 0)

### Out of Scene ###
# SLIDER
scene.append_to_caption('''
<div style="display: inline-block; vertical-align: top; margin-left: 2px; position: relative;">
    <div style="
        position: absolute;
        top: -10px;
        left: 20px;
        width: 300px;
        height: 260px;
        padding: 10px;
        border-radius: 20px;
        background-color: #d4f0f0;
        z-index: -1;">
    </div>
''')

scene.append_to_caption('\n')

gap_slider_text = wtext(text='d(Distance between Plates): {:.1f} mm\n'.format(initial_gap))
scene.append_to_caption('\n\n')
scene.append_to_caption('      ')
gap_slider = slider(min=2.0, max=10.0, value=initial_gap, length=300, bind=update_capacitor)

scene.append_to_caption('\n\n')

area_slider_text = wtext(text='S(Area of Plates): {:.1f} mm^2\n'.format(initial_size))
scene.append_to_caption('\n\n')
scene.append_to_caption('      ')
area_slider = slider(min=10.0, max=20.0, value=initial_size, length=300, bind=update_capacitor)

scene.append_to_caption('\n\n')

dielectric_slider_text = wtext(text='ε(Dielectric Constant): {:.1f}\n'.format(dielectric_constant))
scene.append_to_caption('\n')
scene.append_to_caption('      ')
dielectric_slider = slider(min=1.0, max=10.0, value=dielectric_constant, length=300, bind=update_capacitor)

scene.append_to_caption('\n\n\n\n')

# TEXT
scene.append_to_caption('''
<div style="display: inline-block; vertical-align: top; margin-left: 2px; position: relative;">
    <div style="
        position: absolute;
        top: 0px;
        left: 20px;
        width: 300px;
        height: 100px;
        padding: 10px;
        border-radius: 20px;
        background-color: #d4f0f0;
        z-index: -1;">
    </div>
''')

scene.append_to_caption('\n')

capacitance_text = wtext(text=' C (Capacitance): {:.2e} pF\n'.format(epsilon_0 * initial_size * 2000.0 * 1e-6 / (initial_gap * 1e-3) * 1e12))

scene.append_to_caption('\n\n')

charge = epsilon_0 * initial_size * 2000.0 * 1e-6 / (initial_gap * 1e-3) * voltage
charge_text = wtext(text=' Q (Charge): {:.2e} pC\n'.format(charge * 1e12))

scene.append_to_caption('\n\n\n\n')

# BUTTON
scene.append_to_caption('''
<div style="display: inline-block; vertical-align: top; margin-left: 2px; position: relative;">
    <div style="
        position: absolute;
        top: 0px;
        left: 20px;
        width: 300px;
        height: 70px;
        padding: 10px;
        border-radius: 20px;
        background-color: #d4f0f0;
        z-index: -1;">
    </div>
''')

scene.append_to_caption('\n')

scene.append_to_caption('       ')
button(bind=connect_capacitor_to_led, text=' 💡 Connect LED ')
scene.append_to_caption(' ')
button(bind=reconnect_battery, text=' 🔋 Connect Battery  ')

# GUIDE
scene.append_to_caption('''
<div style="display: inline-block; vertical-align: top; margin-left: 350px; position: relative;">
    <div style="
        position: absolute;
        top: -525px;
        left: 20px;
        width: 500px;
        height: 533px;
        padding: 10px;
        border-radius: 20px;
        background-color: #ffdbcc;
        z-index: -1;">
        <h2 style="text-align: center; margin-top: 0; margin-bottom: 0;">축전기 시뮬레이터에 오신 것을 환영합니다!</h2>


<h3 style="margin-bottom: 0;">       슬라이더</h3><hr style="border: 0; height: 1px; background-color: black; margin: 37px; margin-top: 0; margin-bottom: 0;">        축전기의 두 판 사이 간격 바꾸기
        축전기의 두 판 면적 크기 바꾸기
        유전체의 유전율 크기 바꾸기
        
<h3 style="margin-bottom: 0;">       버튼</h3><hr style="border: 0; height: 1px; background-color: black; margin: 37px; margin-top: 0; margin-bottom: 0;">        축전기를 LED와 연결하기
        축전기를 배터리와 연결하기
        유전체를 드래그 앤 드롭하기
        
<h3 style="margin-bottom: 0;">       기타</h3><hr style="border: 0; height: 1px; background-color: black; margin: 37px; margin-top: 0; margin-bottom: 0;">        유전체를 마우스로 끌어서 축전기와 연결하거나, 연결을 해제해보세요.
        축전기를 배터리와 연결하면 충전됩니다
        축전기를 LED와 연결하면 LED가 켜지며 서서히 어두워집니다
        배터리와의 연결은 전하량이 0이 될때 누르세요
    </div>
</div>
''')

### Initial update ###
update_capacitor()
