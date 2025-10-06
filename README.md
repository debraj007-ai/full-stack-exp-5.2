const express = require("express");
const mongoose = require("mongoose");
const dotenv = require("dotenv");

dotenv.config();
const app = express();
app.use(express.json());
mongoose
  .connect(process.env.MONGO_URI || "mongodb://localhost:27017/studentdb", {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  })
  .then(() => console.log("MongoDB Connected"))
  .catch((err) => console.error("MongoDB connection failed:", err));
const studentSchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: [true, "Student name is required"],
    },
    age: {
      type: Number,
      required: [true, "Student age is required"],
      min: [1, "Age must be greater than 0"],
    },
    course: {
      type: String,
      required: [true, "Course is required"],
    },
  },
  { timestamps: true }
);

const Student = mongoose.model("Student", studentSchema);
app.post("/api/students", async (req, res) => {
  try {
    const { name, age, course } = req.body;
    const student = await Student.create({ name, age, course });
    res.status(201).json({ message: "Student created successfully", student });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

app.get("/api/students", async (req, res) => {
  try {
    const students = await Student.find();
    res.status(200).json(students);
  } catch (error) {
    res.status(500).json({ error: "Failed to fetch students" });
  }
});

app.put("/api/students/:id", async (req, res) => {
  try {
    const { id } = req.params;
    const student = await Student.findByIdAndUpdate(id, req.body, {
      new: true,
      runValidators: true,
    });
    if (!student) return res.status(404).json({ error: "Student not found" });
    res.json({ message: "Student updated successfully", student });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});
app.delete("/api/students/:id", async (req, res) => {
  try {
    const { id } = req.params;
    const student = await Student.findByIdAndDelete(id);
    if (!student) return res.status(404).json({ error: "Student not found" });
    res.json({ message: "Student deleted successfully" });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
app.get("/", (req, res) => {
  res.send("Student Management System API is running...");
});
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(` Server running on port ${PORT}`));
