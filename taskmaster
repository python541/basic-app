#!/usr/bin/env python3
from flask import Flask, jsonify, request, render_template, abort
import json
import os

APP_DIR = os.path.dirname(__file__)
TASKS_FILE = os.path.join(APP_DIR, "tasks.json")

app = Flask(__name__, template_folder="templates")


def load_tasks():
    if not os.path.exists(TASKS_FILE):
        return []
    try:
        with open(TASKS_FILE, "r", encoding="utf-8") as f:
            return json.load(f)
    except Exception:
        return []


def save_tasks(tasks):
    with open(TASKS_FILE, "w", encoding="utf-8") as f:
        json.dump(tasks, f, indent=2)


def next_id(tasks):
    if not tasks:
        return 1
    return max(t["id"] for t in tasks) + 1


@app.route("/")
def index():
    return render_template("index.html")


# API: get tasks
@app.route("/api/tasks", methods=["GET"])
def get_tasks():
    tasks = load_tasks()
    return jsonify(tasks)


# API: create a task
@app.route("/api/tasks", methods=["POST"])
def create_task():
    data = request.get_json(force=True, silent=True)
    if not data or "title" not in data:
        return jsonify({"error": "Missing 'title' in request body"}), 400
    tasks = load_tasks()
    task = {
        "id": next_id(tasks),
        "title": str(data["title"]).strip(),
        "completed": bool(data.get("completed", False))
    }
    tasks.append(task)
    save_tasks(tasks)
    return jsonify(task), 201


# API: update a task (partial)
@app.route("/api/tasks/<int:task_id>", methods=["PUT", "PATCH"])
def update_task(task_id):
    data = request.get_json(force=True, silent=True)
    if data is None:
        return jsonify({"error": "Missing JSON body"}), 400
    tasks = load_tasks()
    for t in tasks:
        if t["id"] == task_id:
            if "title" in data:
                t["title"] = str(data["title"]).strip()
            if "completed" in data:
                t["completed"] = bool(data["completed"])
            save_tasks(tasks)
            return jsonify(t)
    return jsonify({"error": "Task not found"}), 404


# API: delete a task
@app.route("/api/tasks/<int:task_id>", methods=["DELETE"])
def delete_task(task_id):
    tasks = load_tasks()
    new_tasks = [t for t in tasks if t["id"] != task_id]
    if len(new_tasks) == len(tasks):
        return jsonify({"error": "Task not found"}), 404
    save_tasks(new_tasks)
    return jsonify({"deleted": task_id})


if __name__ == "__main__":
    # Create tasks file if missing
    if not os.path.exists(TASKS_FILE):
        save_tasks([])
    app.run(host="0.0.0.0", port=5000, debug=True)
