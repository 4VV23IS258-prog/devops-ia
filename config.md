# Generated Code Explained With Code

This version includes the generated code directly and explains each file without line numbers.

## requirements.txt

```python
pygame>=2.5.0
pynput>=1.7.6
pywin32>=306
```

What this does:
- Installs pygame for controller input and joystick polling.
- Installs pynput for mouse movement, clicking, and scrolling actions.
- Installs pywin32 for Windows-specific support and compatibility.

## src/main.py

```python
from __future__ import annotations

import ctypes
import sys
import tkinter as tk

from app import KeyMapperApp


def make_dpi_aware() -> None:
	if sys.platform.startswith("win"):
		try:
			ctypes.windll.user32.SetProcessDPIAware()
		except OSError:
			pass


def main() -> None:
	make_dpi_aware()
	root = tk.Tk()
	KeyMapperApp(root)
	root.mainloop()


if __name__ == "__main__":
	main()
```

What this does:
- Creates the app entry point.
- Sets Windows DPI awareness so picked points and cursor coordinates align better on scaled displays.
- Starts Tkinter and launches KeyMapperApp.

## src/config.py

```python
from __future__ import annotations

from dataclasses import asdict, dataclass, field
from pathlib import Path
from typing import Any
import json


BUTTON_ACTION_NAMES = [
	"A",
	"B",
	"X",
	"Y",
	"XB",
	"YB",
	"X Trigger",
	"Y Trigger",
]


MOUSE_ACTIONS = [
	"move_click_left",
	"move_click_right",
	"double_click_left",
	"press_left",
	"release_left",
	"scroll_up",
	"scroll_down",
]


@dataclass
class Point:
	x: int
	y: int


@dataclass
class ControllerBinding:
	kind: str = "button"
	index: int = 0
	direction: str = "positive"
	threshold: float = 0.5


@dataclass
class ButtonMapping:
	name: str
	binding: ControllerBinding | None = None
	mouse_action: str = "move_click_left"
	target_point: Point | None = None


@dataclass
class LeftStickConfig:
	center_point: Point | None = None
	radius_point: Point | None = None
	deadzone: float = 0.18


@dataclass
class AppConfig:
	button_mappings: dict[str, ButtonMapping] = field(default_factory=dict)
	left_stick: LeftStickConfig = field(default_factory=LeftStickConfig)


def default_config() -> AppConfig:
	return AppConfig(
		button_mappings={
			name: ButtonMapping(name=name) for name in BUTTON_ACTION_NAMES
		}
	)


def point_from_dict(data: dict[str, Any] | None) -> Point | None:
	if not data:
		return None
	if "x" not in data or "y" not in data:
		return None
	return Point(x=int(data["x"]), y=int(data["y"]))


def binding_from_dict(data: dict[str, Any] | None) -> ControllerBinding | None:
	if not data:
		return None
	return ControllerBinding(
		kind=str(data.get("kind", "button")),
		index=int(data.get("index", 0)),
		direction=str(data.get("direction", "positive")),
		threshold=float(data.get("threshold", 0.5)),
	)


def mapping_from_dict(data: dict[str, Any]) -> ButtonMapping:
	return ButtonMapping(
		name=str(data.get("name", "")),
		binding=binding_from_dict(data.get("binding")),
		mouse_action=str(data.get("mouse_action", "move_click_left")),
		target_point=point_from_dict(data.get("target_point")),
	)


def config_from_dict(data: dict[str, Any] | None) -> AppConfig:
	if not data:
		return default_config()

	mappings_data = data.get("button_mappings", {})
	mappings: dict[str, ButtonMapping] = {}
	for name in BUTTON_ACTION_NAMES:
		mapping_data = mappings_data.get(name)
		if mapping_data:
			mappings[name] = mapping_from_dict(mapping_data)
		else:
			mappings[name] = ButtonMapping(name=name)

	left_stick_data = data.get("left_stick", {})
	left_stick = LeftStickConfig(
		center_point=point_from_dict(left_stick_data.get("center_point")),
		radius_point=point_from_dict(left_stick_data.get("radius_point")),
		deadzone=float(left_stick_data.get("deadzone", 0.18)),
	)

	return AppConfig(button_mappings=mappings, left_stick=left_stick)


def config_to_dict(config: AppConfig) -> dict[str, Any]:
	data = {
		"button_mappings": {},
		"left_stick": {
			"center_point": asdict(config.left_stick.center_point)
			if config.left_stick.center_point
			else None,
			"radius_point": asdict(config.left_stick.radius_point)
			if config.left_stick.radius_point
			else None,
			"deadzone": config.left_stick.deadzone,
		},
	}

	for name, mapping in config.button_mappings.items():
		data["button_mappings"][name] = {
			"name": mapping.name,
			"binding": asdict(mapping.binding) if mapping.binding else None,
			"mouse_action": mapping.mouse_action,
			"target_point": asdict(mapping.target_point) if mapping.target_point else None,
		}

	return data


def load_config(path: Path) -> AppConfig:
	if not path.exists():
		return default_config()

	try:
		with path.open("r", encoding="utf-8") as handle:
			payload = json.load(handle)
	except (OSError, ValueError, json.JSONDecodeError):
		return default_config()

	return config_from_dict(payload)


def save_config(path: Path, config: AppConfig) -> None:
	path.parent.mkdir(parents=True, exist_ok=True)
	with path.open("w", encoding="utf-8") as handle:
		json.dump(config_to_dict(config), handle, indent=2)
```

What this does:
- Defines all config models used by the app.
- Stores your required mapping rows: A, B, X, Y, XB, YB, X Trigger, Y Trigger.
- Stores left stick center/radius/deadzone settings.
- Converts between Python dataclasses and JSON.
- Loads defaults when config.json is missing or invalid.

## src/point_picker.py

```python
from __future__ import annotations

from dataclasses import dataclass
from typing import Callable
import tkinter as tk

from config import Point


@dataclass
class PointPicker:
	root: tk.Tk
	title: str
	on_pick: Callable[[Point | None], None]

	def open(self) -> None:
		window = tk.Toplevel(self.root)
		window.title(self.title)
		window.configure(bg="#101420")
		window.attributes("-topmost", True)
		window.attributes("-fullscreen", True)
		window.attributes("-alpha", 0.92)
		window.configure(cursor="crosshair")

		instruction = tk.Label(
			window,
			text="Click anywhere on the screen to capture that point. Press Esc to cancel.",
			fg="#f4f7fb",
			bg="#101420",
			font=("Segoe UI", 18, "bold"),
			padx=24,
			pady=24,
		)
		instruction.pack(anchor="n")

		hint = tk.Label(
			window,
			text=self.title,
			fg="#9dd3ff",
			bg="#101420",
			font=("Segoe UI", 14),
			padx=24,
			pady=8,
		)
		hint.pack(anchor="n")

		def close_with_result(point: Point | None) -> None:
			try:
				window.grab_release()
			except tk.TclError:
				pass
			try:
				window.destroy()
			finally:
				self.on_pick(point)

		def on_click(event: tk.Event) -> None:
			close_with_result(Point(x=event.x_root, y=event.y_root))

		def on_cancel(_event: tk.Event | None = None) -> None:
			close_with_result(None)

		window.bind("<Button-1>", on_click)
		window.bind("<Escape>", on_cancel)
		window.protocol("WM_DELETE_WINDOW", on_cancel)
		window.focus_force()
		window.grab_set()
```

What this does:
- Creates a fullscreen overlay to capture exact screen coordinates.
- Returns a Point when clicked.
- Returns None when canceled via Escape or close button.

## src/app.py

```python
from __future__ import annotations

from dataclasses import dataclass
from math import hypot
from pathlib import Path
import copy
import queue
import threading
import time
import tkinter as tk
from tkinter import ttk

import pygame
from pynput.mouse import Button, Controller

from config import (
	AppConfig,
	BUTTON_ACTION_NAMES,
	ControllerBinding,
	MOUSE_ACTIONS,
	Point,
	default_config,
	load_config,
	save_config,
)
from point_picker import PointPicker


def format_point(point: Point | None) -> str:
	if point is None:
		return "Not set"
	return f"({point.x}, {point.y})"


def format_binding(binding: ControllerBinding | None) -> str:
	if binding is None:
		return "Unbound"
	if binding.kind == "axis":
		return f"Axis {binding.index} {binding.direction}"
	return f"Button {binding.index}"


def binding_to_text(binding: ControllerBinding | None) -> str:
	return format_binding(binding)


@dataclass
class RuntimeState:
	capture_target: str | None = None
	joystick_name: str | None = None
	joystick_connected: bool = False


class InputWorker(threading.Thread):
	def __init__(
		self,
		config: AppConfig,
		config_lock: threading.Lock,
		state: RuntimeState,
		state_lock: threading.Lock,
		event_queue: queue.Queue,
		stop_event: threading.Event,
		mouse: Controller,
	) -> None:
		super().__init__(daemon=True)
		self._config = config
		self._config_lock = config_lock
		self._state = state
		self._state_lock = state_lock
		self._event_queue = event_queue
		self._stop_event = stop_event
		self._mouse = mouse
		self._joystick: pygame.joystick.Joystick | None = None
		self._prev_buttons: list[bool] = []
		self._prev_axes: list[float] = []
		self._dragging = False
		self._last_status = ""

	def run(self) -> None:
		pygame.init()
		pygame.joystick.init()

		while not self._stop_event.is_set():
			try:
				pygame.event.pump()
			except pygame.error:
				pass

			self._ensure_joystick()
			snapshot = self._snapshot_config()
			capture_target = self._get_capture_target()

			if self._joystick is None:
				self._set_status_once("Waiting for controller...")
				time.sleep(0.1)
				continue

			self._set_status_once(f"Controller connected: {self._joystick.get_name()}")

			buttons = self._read_buttons()
			axes = self._read_axes()

			if capture_target:
				captured = self._detect_capture(buttons, axes)
				if captured is not None:
					self._event_queue.put(("captured_input", capture_target, captured))
					self._clear_capture_target()
			else:
				self._process_mappings(snapshot, buttons, axes)

			self._process_left_stick(snapshot, axes)
			self._prev_buttons = buttons
			self._prev_axes = axes
			time.sleep(0.01)

		try:
			pygame.joystick.quit()
			pygame.quit()
		except pygame.error:
			pass

	def _ensure_joystick(self) -> None:
		if self._joystick is not None and self._joystick.get_init():
			return

		count = pygame.joystick.get_count()
		if count <= 0:
			self._joystick = None
			self._prev_buttons = []
			self._prev_axes = []
			self._set_connection_state(False, None)
			return

		joystick = pygame.joystick.Joystick(0)
		joystick.init()
		self._joystick = joystick
		self._prev_buttons = [False] * joystick.get_numbuttons()
		self._prev_axes = [0.0] * joystick.get_numaxes()
		self._set_connection_state(True, joystick.get_name())

	def _read_buttons(self) -> list[bool]:
		if self._joystick is None:
			return []
		total = self._joystick.get_numbuttons()
		return [bool(self._joystick.get_button(index)) for index in range(total)]

	def _read_axes(self) -> list[float]:
		if self._joystick is None:
			return []
		total = self._joystick.get_numaxes()
		return [float(self._joystick.get_axis(index)) for index in range(total)]

	def _snapshot_config(self) -> AppConfig:
		with self._config_lock:
			return copy.deepcopy(self._config)

	def _get_capture_target(self) -> str | None:
		with self._state_lock:
			return self._state.capture_target

	def _clear_capture_target(self) -> None:
		with self._state_lock:
			self._state.capture_target = None

	def _set_status_once(self, message: str) -> None:
		if message == self._last_status:
			return
		self._last_status = message
		self._event_queue.put(("status", message))

	def _set_connection_state(self, connected: bool, name: str | None) -> None:
		with self._state_lock:
			self._state.joystick_connected = connected
			self._state.joystick_name = name

	def _detect_capture(self, buttons: list[bool], axes: list[float]) -> ControllerBinding | None:
		button_count = min(len(buttons), len(self._prev_buttons))
		for index in range(button_count):
			if buttons[index] and not self._prev_buttons[index]:
				return ControllerBinding(kind="button", index=index)

		axis_count = min(len(axes), len(self._prev_axes))
		for index in range(axis_count):
			current = axes[index]
			previous = self._prev_axes[index]
			if abs(current) >= 0.5 and abs(previous) < 0.5:
				direction = "positive" if current >= 0 else "negative"
				return ControllerBinding(kind="axis", index=index, direction=direction, threshold=0.5)

		return None

	def _process_mappings(self, config: AppConfig, buttons: list[bool], axes: list[float]) -> None:
		for mapping in config.button_mappings.values():
			binding = mapping.binding
			if binding is None:
				continue

			triggered = False
			if binding.kind == "button":
				if binding.index < len(buttons) and binding.index < len(self._prev_buttons):
					triggered = buttons[binding.index] and not self._prev_buttons[binding.index]
			elif binding.kind == "axis":
				if binding.index < len(axes) and binding.index < len(self._prev_axes):
					current = axes[binding.index]
					previous = self._prev_axes[binding.index]
					if binding.direction == "positive":
						triggered = current >= binding.threshold and previous < binding.threshold
					else:
						triggered = current <= -binding.threshold and previous > -binding.threshold

			if triggered:
				self._execute_mouse_action(mapping.mouse_action, mapping.target_point)

	def _process_left_stick(self, config: AppConfig, axes: list[float]) -> None:
		left_stick = config.left_stick
		if left_stick.center_point is None or left_stick.radius_point is None:
			self._release_drag_if_needed()
			return

		x_axis = axes[0] if len(axes) > 0 else 0.0
		y_axis = axes[1] if len(axes) > 1 else 0.0
		magnitude = hypot(x_axis, y_axis)
		if magnitude <= left_stick.deadzone:
			self._release_drag_if_needed()
			return

		radius = hypot(
			left_stick.radius_point.x - left_stick.center_point.x,
			left_stick.radius_point.y - left_stick.center_point.y,
		)
		target_x = int(round(left_stick.center_point.x + x_axis * radius))
		target_y = int(round(left_stick.center_point.y + y_axis * radius))

		if not self._dragging:
			self._mouse.position = (left_stick.center_point.x, left_stick.center_point.y)
			self._mouse.press(Button.left)
			self._dragging = True

		self._mouse.position = (target_x, target_y)

	def _release_drag_if_needed(self) -> None:
		if not self._dragging:
			return
		try:
			self._mouse.release(Button.left)
		finally:
			self._dragging = False

	def _execute_mouse_action(self, action: str, point: Point | None) -> None:
		if action in {"move_click_left", "move_click_right", "double_click_left", "press_left", "release_left"}:
			if point is None:
				self._event_queue.put(("status", f"Skipped {action}: target point not set"))
				return
			self._mouse.position = (point.x, point.y)

		if action == "move_click_left":
			self._mouse.click(Button.left, 1)
		elif action == "move_click_right":
			self._mouse.click(Button.right, 1)
		elif action == "double_click_left":
			self._mouse.click(Button.left, 2)
		elif action == "press_left":
			self._mouse.press(Button.left)
		elif action == "release_left":
			self._mouse.release(Button.left)
		elif action == "scroll_up":
			self._mouse.scroll(0, 2)
		elif action == "scroll_down":
			self._mouse.scroll(0, -2)


class KeyMapperApp:
	def __init__(self, root: tk.Tk) -> None:
		self.root = root
		self.root.title("Keymapper")
		self.root.geometry("1200x820")
		self.root.minsize(1080, 720)

		self.config_path = Path(__file__).resolve().parent.parent / "config.json"
		self.config = load_config(self.config_path)
		if not self.config.button_mappings:
			self.config = default_config()

		self.config_lock = threading.Lock()
		self.state = RuntimeState()
		self.state_lock = threading.Lock()
		self.event_queue: queue.Queue = queue.Queue()
		self.stop_event = threading.Event()
		self.mouse = Controller()
		self.worker = InputWorker(
			config=self.config,
			config_lock=self.config_lock,
			state=self.state,
			state_lock=self.state_lock,
			event_queue=self.event_queue,
			stop_event=self.stop_event,
			mouse=self.mouse,
		)

		self.status_var = tk.StringVar(value="Starting...")
		self.joystick_var = tk.StringVar(value="No controller detected yet")
		self.deadzone_var = tk.DoubleVar(value=self.config.left_stick.deadzone)
		self.mapping_rows: dict[str, dict[str, tk.Variable | ttk.Combobox | tk.Label]] = {}

		self._build_ui()
		self.worker.start()
		self.root.after(50, self._poll_events)
		self.root.protocol("WM_DELETE_WINDOW", self.close)

	def _build_ui(self) -> None:
		outer = ttk.Frame(self.root, padding=16)
		outer.pack(fill="both", expand=True)

		title = ttk.Label(outer, text="Keymapper", font=("Segoe UI", 22, "bold"))
		title.pack(anchor="w")

		subtitle = ttk.Label(
			outer,
			text="Bind controller inputs at runtime, pick mouse targets on screen, and map the left stick to drag movement.",
			font=("Segoe UI", 10),
		)
		subtitle.pack(anchor="w", pady=(4, 12))

		status_bar = ttk.Frame(outer)
		status_bar.pack(fill="x", pady=(0, 12))
		ttk.Label(status_bar, textvariable=self.status_var).pack(side="left")
		ttk.Label(status_bar, textvariable=self.joystick_var).pack(side="right")

		toolbar = ttk.Frame(outer)
		toolbar.pack(fill="x", pady=(0, 14))
		ttk.Button(toolbar, text="Save Config", command=self.save_current_config).pack(side="left")
		ttk.Button(toolbar, text="Reload Config", command=self.reload_config).pack(side="left", padx=(8, 0))

		notebook = ttk.Notebook(outer)
		notebook.pack(fill="both", expand=True)

		buttons_tab = ttk.Frame(notebook, padding=12)
		stick_tab = ttk.Frame(notebook, padding=12)
		notebook.add(buttons_tab, text="Buttons")
		notebook.add(stick_tab, text="Left Stick")

		self._build_buttons_tab(buttons_tab)
		self._build_left_stick_tab(stick_tab)
		self._refresh_ui_from_config()

	def _build_buttons_tab(self, parent: ttk.Frame) -> None:
		header = ttk.Frame(parent)
		header.pack(fill="x", pady=(0, 8))
		ttk.Label(header, text="Controller Button Mapping", font=("Segoe UI", 13, "bold")).pack(anchor="w")
		ttk.Label(
			header,
			text="Use Bind to learn the controller input, Pick Point to set the screen target, and choose the mouse action.",
		).pack(anchor="w", pady=(4, 0))

		rows = ttk.Frame(parent)
		rows.pack(fill="both", expand=True)
		rows.columnconfigure(2, weight=1)

		ttk.Label(rows, text="Action", font=("Segoe UI", 10, "bold")).grid(row=0, column=0, sticky="w", padx=4, pady=6)
		ttk.Label(rows, text="Controller Input", font=("Segoe UI", 10, "bold")).grid(row=0, column=1, sticky="w", padx=4, pady=6)
		ttk.Label(rows, text="Mouse Action", font=("Segoe UI", 10, "bold")).grid(row=0, column=2, sticky="w", padx=4, pady=6)
		ttk.Label(rows, text="Target Point", font=("Segoe UI", 10, "bold")).grid(row=0, column=3, sticky="w", padx=4, pady=6)

		for index, name in enumerate(BUTTON_ACTION_NAMES, start=1):
			mapping = self.config.button_mappings[name]
			ttk.Label(rows, text=name, width=16).grid(row=index, column=0, sticky="w", padx=4, pady=6)

			binding_var = tk.StringVar(value=binding_to_text(mapping.binding))
			ttk.Label(rows, textvariable=binding_var, width=22).grid(row=index, column=1, sticky="w", padx=4, pady=6)

			action_var = tk.StringVar(value=mapping.mouse_action)
			action_box = ttk.Combobox(rows, textvariable=action_var, values=MOUSE_ACTIONS, state="readonly", width=24)
			action_box.grid(row=index, column=2, sticky="we", padx=4, pady=6)
			action_box.bind(
				"<<ComboboxSelected>>",
				lambda _event, action_name=name, variable=action_var: self._on_mouse_action_changed(action_name, variable.get()),
			)

			point_var = tk.StringVar(value=format_point(mapping.target_point))
			ttk.Label(rows, textvariable=point_var, width=20).grid(row=index, column=3, sticky="w", padx=4, pady=6)

			button_frame = ttk.Frame(rows)
			button_frame.grid(row=index, column=4, sticky="e", padx=4, pady=6)
			ttk.Button(button_frame, text="Bind", command=lambda action_name=name: self.begin_capture(action_name)).pack(side="left")
			ttk.Button(button_frame, text="Pick Point", command=lambda action_name=name: self.pick_action_point(action_name)).pack(side="left", padx=(8, 0))
			ttk.Button(button_frame, text="Clear", command=lambda action_name=name: self.clear_action(action_name)).pack(side="left", padx=(8, 0))

			self.mapping_rows[name] = {
				"binding_var": binding_var,
				"action_var": action_var,
				"point_var": point_var,
			}

	def _build_left_stick_tab(self, parent: ttk.Frame) -> None:
		top = ttk.Frame(parent)
		top.pack(fill="x")
		ttk.Label(top, text="Left Stick Motion", font=("Segoe UI", 13, "bold")).pack(anchor="w")
		ttk.Label(
			top,
			text="Pick a center point and a second point to define the radius. The stick then drags from that center using the stick deflection.",
		).pack(anchor="w", pady=(4, 10))

		controls = ttk.Frame(parent)
		controls.pack(fill="x", pady=(4, 12))
		controls.columnconfigure(1, weight=1)

		ttk.Label(controls, text="Center Point").grid(row=0, column=0, sticky="w", padx=4, pady=6)
		self.center_var = tk.StringVar(value=format_point(self.config.left_stick.center_point))
		ttk.Label(controls, textvariable=self.center_var).grid(row=0, column=1, sticky="w", padx=4, pady=6)
		ttk.Button(controls, text="Pick Center", command=self.pick_stick_center).grid(row=0, column=2, padx=4, pady=6)

		ttk.Label(controls, text="Radius Point").grid(row=1, column=0, sticky="w", padx=4, pady=6)
		self.radius_var = tk.StringVar(value=format_point(self.config.left_stick.radius_point))
		ttk.Label(controls, textvariable=self.radius_var).grid(row=1, column=1, sticky="w", padx=4, pady=6)
		ttk.Button(controls, text="Pick Radius", command=self.pick_stick_radius).grid(row=1, column=2, padx=4, pady=6)

		ttk.Label(controls, text="Deadzone").grid(row=2, column=0, sticky="w", padx=4, pady=6)
		deadzone_scale = ttk.Scale(controls, from_=0.0, to=0.5, variable=self.deadzone_var, command=self._on_deadzone_changed)
		deadzone_scale.grid(row=2, column=1, sticky="we", padx=4, pady=6)
		self.deadzone_label_var = tk.StringVar(value=f"{self.deadzone_var.get():.2f}")
		ttk.Label(controls, textvariable=self.deadzone_label_var).grid(row=2, column=2, sticky="w", padx=4, pady=6)

		note = ttk.Label(
			parent,
			text="When the stick leaves the deadzone, the app holds left mouse down, moves the cursor from the chosen center, and releases on stick return.",
			wraplength=980,
		)
		note.pack(anchor="w", pady=(10, 0))

	def _on_mouse_action_changed(self, action_name: str, action_value: str) -> None:
		with self.config_lock:
			self.config.button_mappings[action_name].mouse_action = action_value
			save_config(self.config_path, self.config)
		self._refresh_ui_from_config()

	def _on_deadzone_changed(self, _value: str) -> None:
		value = float(self.deadzone_var.get())
		self.deadzone_label_var.set(f"{value:.2f}")
		with self.config_lock:
			self.config.left_stick.deadzone = value
			save_config(self.config_path, self.config)

	def begin_capture(self, action_name: str) -> None:
		with self.state_lock:
			self.state.capture_target = action_name
		self.status_var.set(f"Waiting for controller input for {action_name}...")

	def pick_action_point(self, action_name: str) -> None:
		PointPicker(self.root, f"Pick point for {action_name}", lambda point: self._set_action_point(action_name, point)).open()

	def _set_action_point(self, action_name: str, point: Point | None) -> None:
		if point is None:
			self.status_var.set(f"Cancelled point pick for {action_name}")
			return
		with self.config_lock:
			self.config.button_mappings[action_name].target_point = point
			save_config(self.config_path, self.config)
		self.status_var.set(f"Set {action_name} target to ({point.x}, {point.y})")
		self._refresh_ui_from_config()

	def clear_action(self, action_name: str) -> None:
		with self.config_lock:
			self.config.button_mappings[action_name].binding = None
			self.config.button_mappings[action_name].target_point = None
			self.config.button_mappings[action_name].mouse_action = "move_click_left"
			save_config(self.config_path, self.config)
		self.status_var.set(f"Cleared {action_name}")
		self._refresh_ui_from_config()

	def pick_stick_center(self) -> None:
		PointPicker(self.root, "Pick left stick center", self._set_stick_center).open()

	def _set_stick_center(self, point: Point | None) -> None:
		if point is None:
			self.status_var.set("Cancelled stick center pick")
			return
		with self.config_lock:
			self.config.left_stick.center_point = point
			save_config(self.config_path, self.config)
		self.status_var.set(f"Stick center set to ({point.x}, {point.y})")
		self._refresh_ui_from_config()

	def pick_stick_radius(self) -> None:
		PointPicker(self.root, "Pick left stick radius point", self._set_stick_radius).open()

	def _set_stick_radius(self, point: Point | None) -> None:
		if point is None:
			self.status_var.set("Cancelled stick radius pick")
			return
		with self.config_lock:
			self.config.left_stick.radius_point = point
			save_config(self.config_path, self.config)
		self.status_var.set(f"Stick radius point set to ({point.x}, {point.y})")
		self._refresh_ui_from_config()

	def save_current_config(self) -> None:
		with self.config_lock:
			save_config(self.config_path, self.config)
		self.status_var.set(f"Saved config to {self.config_path.name}")

	def reload_config(self) -> None:
		loaded = load_config(self.config_path)
		if not loaded.button_mappings:
			loaded = default_config()
		with self.config_lock:
			self.config.button_mappings.clear()
			self.config.button_mappings.update(loaded.button_mappings)
			self.config.left_stick.center_point = loaded.left_stick.center_point
			self.config.left_stick.radius_point = loaded.left_stick.radius_point
			self.config.left_stick.deadzone = loaded.left_stick.deadzone
		self.status_var.set("Reloaded config from disk")
		self._refresh_ui_from_config()

	def _refresh_ui_from_config(self) -> None:
		with self.config_lock:
			config = copy.deepcopy(self.config)

		self.center_var.set(format_point(config.left_stick.center_point))
		self.radius_var.set(format_point(config.left_stick.radius_point))
		self.deadzone_var.set(config.left_stick.deadzone)
		self.deadzone_label_var.set(f"{config.left_stick.deadzone:.2f}")

		for name, row in self.mapping_rows.items():
			mapping = config.button_mappings[name]
			row["binding_var"].set(binding_to_text(mapping.binding))
			row["action_var"].set(mapping.mouse_action)
			row["point_var"].set(format_point(mapping.target_point))

	def _poll_events(self) -> None:
		while True:
			try:
				event_type, *payload = self.event_queue.get_nowait()
			except queue.Empty:
				break

			if event_type == "status":
				self.status_var.set(str(payload[0]))
			elif event_type == "captured_input":
				action_name = str(payload[0])
				binding = payload[1]
				self._apply_captured_binding(action_name, binding)
			elif event_type == "controller":
				self.joystick_var.set(str(payload[0]))

		with self.state_lock:
			if self.state.joystick_connected and self.state.joystick_name:
				self.joystick_var.set(f"Connected: {self.state.joystick_name}")
			else:
				self.joystick_var.set("No controller detected yet")

		self.root.after(50, self._poll_events)

	def _apply_captured_binding(self, action_name: str, binding: ControllerBinding) -> None:
		with self.config_lock:
			self.config.button_mappings[action_name].binding = binding
			save_config(self.config_path, self.config)
		self.status_var.set(f"Bound {action_name} to {format_binding(binding)}")
		self._refresh_ui_from_config()

	def close(self) -> None:
		self.stop_event.set()
		try:
			self.worker.join(timeout=1.5)
		except RuntimeError:
			pass
		self.root.destroy()
```

What this does:
- Builds the full Tkinter UI and manages runtime interaction.
- Starts a background worker that reads gamepad state continuously.
- Lets you bind each required action row by pressing controller inputs.
- Lets you pick a target screen point per action row.
- Executes action-based mouse behavior when mapped rows trigger.
- Implements left-stick drag behavior:
  - uses picked center point as drag start
  - uses picked radius point to compute max movement radius
  - presses and holds left click when stick exits deadzone
  - moves cursor according to stick direction
  - releases left click when stick returns to deadzone
- Persists config in config.json and supports save/reload from UI.
