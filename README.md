# WordHunt Solver

A AI-powered macOS application that automatically solves GamePigeon's Word Hunt puzzles using computer vision and deep learning. Finds all possible words and swipes them for you.

**World Record Achievement**: This solver achieved the world record highest score of **362,000 points** and the highest recorded score (captured on video) of **299,000 points**.

[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org/)
[![PyTorch](https://img.shields.io/badge/PyTorch-2.1+-orange.svg)](https://pytorch.org/)
[![macOS](https://img.shields.io/badge/macOS-Sonoma%2015.0%2B-lightgrey.svg)](https://www.apple.com/macos/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

## Prerequisites

Before installing, ensure you have:

- **macOS Sonoma (15.0) or later** - Required for iPhone Mirroring feature
- **MacBook with iPhone Mirroring app enabled**
- **iPhone and MacBook logged into the same iCloud account** - Required for iPhone Mirroring to work
- **Screen Recording and Accessibility permissions** - Will be set up during first launch

## Features

- **AI-Powered Letter Recognition**: Custom CNN model trained on thousands of letter images for accurate OCR
- **Automatic Game Solving**: Detects the board, finds all valid words, and swipes them automatically
- **Real-Time Dashboard**: Cocoa GUI showing board preview, word progress, and statistics
- **Smart Word Finding**: Uses Trie data structure for efficient word search across the 4×4 grid
- **Calibration System**: One-time calibration for perfect board detection accuracy
- **Developer Tools**: Scripts for dataset creation and model training

## Demo

Watch the solver in action, achieving a **299,000 point score** (highest recorded score on video):

[![WordHunt Solver Demo](https://img.youtube.com/vi/t09wYAGTHrw/maxresdefault.jpg)](https://youtu.be/t09wYAGTHrw)

The solver captures the game board from iPhone Mirroring, recognizes letters using a trained CNN, finds all valid words using a Trie-based search, and automatically swipes them in optimal order. This implementation has achieved the world record highest score of **362,000 points**.

## How It Works

### Architecture Overview

```
┌─────────────────┐
│ iPhone Mirroring│
│   (Game Board)  │
└────────┬────────┘
         │ Screen Capture (mss)
         ▼
┌─────────────────┐
│  Board Detection│ ← OpenCV (green mask, contour detection)
│   (4×4 Grid)    │
└────────┬────────┘
         │ Extract Cells
         ▼
┌─────────────────┐
│ Letter Recognition│ ← CNN Model (PyTorch)
│   (A-Z, 26 classes)│
└────────┬────────┘
         │ Letters Array
         ▼
┌─────────────────┐
│  Word Finding    │ ← Trie + DFS Search
│  (All valid words)│
└────────┬────────┘
         │ Word List (sorted by length)
         ▼
┌─────────────────┐
│  Auto Swiping    │ ← Quartz CGEvent (mouse simulation)
│  (Input words)   │
└─────────────────┘
```

### Components

1. **Board Detection** (`wordhunt.py`)
   - Uses OpenCV to detect the green game board background
   - Applies morphological operations to find board contours
   - Calculates precise 4×4 grid cell positions
   - Supports calibration for improved accuracy

2. **Letter Recognition** (`wordhunt.py` + `letter_recognition_model.pth`)
   - Custom CNN architecture (2 conv layers + 2 FC layers)
   - Trained on 2,400+ labeled letter images
   - Preprocesses cell images (grayscale, resize to 28×28, normalize)
   - Returns letter predictions with confidence scores

3. **Word Finding** (`wordhunt.py`)
   - Builds Trie from dictionary (handles 'QU' tiles correctly)
   - Depth-first search across all 8 directions
   - Validates words against dictionary
   - Sorts by length (longest first) for maximum points

4. **Automation** (`wordhunt.py`)
   - Uses Quartz CGEvent API for precise mouse control
   - Calculates swipe paths through grid cells
   - Handles coordinate system conversions (window ↔ screen ↔ Quartz)
   - Respects game timing constraints (1:15 round limit)

5. **GUI** (`app_cocoa.py`)
   - Native macOS Cocoa application
   - Real-time board preview
   - Word progress tracking
   - Logs and diagnostics

## Model Details

### Architecture

The letter recognition model is a Convolutional Neural Network with the following structure:

```
Input: 28×28 grayscale image
  ↓
Conv2d(1 → 32, kernel=3) + ReLU
  ↓
MaxPool2d(2×2)
  ↓
Conv2d(32 → 64, kernel=3) + ReLU
  ↓
MaxPool2d(2×2)
  ↓
Flatten → Linear(64×5×5 → 128) + ReLU
  ↓
Dropout(0.5)
  ↓
Linear(128 → 26) + LogSoftmax
  ↓
Output: 26 classes (A-Z)
```

### Training Process

1. **Dataset Creation** (`create_dataset.py`)
   - Captures game boards from iPhone Mirroring
   - Extracts individual cell images
   - Manual labeling interface with AI suggestions
   - Deduplication by image hash
   - Organizes images into `letter_dataset/A/`, `letter_dataset/B/`, etc.

2. **Model Training** (`train_model.py`)
   - 80/20 train/validation split
   - Adam optimizer with CrossEntropyLoss
   - Saves best model based on validation accuracy
   - Provides per-letter diagnostics for problematic classes
   - Typical training: 50-100 epochs

3. **Model Performance**
   - Trained on 2,400+ labeled images
   - High accuracy on letter recognition
   - Handles various lighting conditions and board states
   - Model file: `letter_recognition_model.pth` (896 KB)

### Dataset Statistics

- **Total Images**: 2,407 labeled letter samples
- **Distribution**: Varies by letter frequency (E, A, R most common; Q, Z least common)
- **Format**: 28×28 grayscale PNG images
- **Organization**: One folder per letter (A-Z)

## Installation

### Option 1: Download Pre-built Release (Recommended)

1. **Download the latest release**
   - Go to the [Releases](https://github.com/Ammaar-Alam/wordhunt-solver/releases) page
   - Download `WordHuntSolver-macOS.zip` from the latest release

2. **Extract and open the app**
   - Extract the ZIP file
   - Double-click `WordHuntSolver.app` to open it
   - macOS will show a security warning: "WordHuntSolver cannot be opened because the developer cannot be verified"
   - Click **Cancel**

3. **Trust and open the app**
   - Open **System Settings** → **Privacy & Security**
   - Scroll down to find the message: "WordHuntSolver was blocked from use because it is not from an identified developer"
   - Click **Open Anyway** next to the message
   - Confirm by clicking **Open** in the dialog

4. **Set up permissions**
   - When the app opens, click the **"Permissions"** button in the app
   - This will open System Settings to the relevant privacy sections
   - Enable **Screen Recording** permission
   - Enable **Accessibility** permission
   - You may need to **restart the app** after granting permissions

5. **You're ready to go!**
   - Open iPhone Mirroring with Word Hunt game
   - Click "Move to Corner" to position the window
   - Send the Word Hunt text, and open it to the Start screen
   - Click "Start" on the app to begin solving

### Option 2: Install from Source

1. **Clone the repository**
   ```bash
   git clone https://github.com/Ammaar-Alam/wordhunt-solver.git
   cd wordhunt-solver
   ```

2. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

3. **Run the application**
   ```bash
   python app_cocoa.py
   ```

4. **Set up permissions**
   - Click the **"Permissions"** button in the app
   - Enable Screen Recording and Accessibility permissions
   - Restart the app if needed

## Usage

### GUI Mode (Recommended)

1. **Launch the app**
   - If installed from release: Open `WordHuntSolver.app`
   - If installed from source: Run `python app_cocoa.py`

2. **Initial setup**
   - Click **"Permissions"** button to open System Settings
   - Grant **Screen Recording** permission
   - Grant **Accessibility** permission
   - **Restart the app** if permissions were just granted

3. **Prepare iPhone Mirroring**
   - Open iPhone Mirroring on your MacBook
   - Open Word Hunt game on your iPhone
   - Click **"Move to Corner"** in the app to position the iPhone Mirroring window

4. **Calibration** (Optional, recommended for first use)
   - Click **"Calibrate"**
   - Hover over center of top-left tile, wait for countdown
   - Hover over center of bottom-right tile, wait for countdown
   - Calibration is saved for future runs

5. **Solve**
   - Click **"Start"** to begin solving
   - Watch the dashboard for progress
   - Words are swiped automatically in optimal order

### Command Line Mode

```bash
# Interactive menu
python wordhunt.py menu

# Auto-solve (non-interactive)
python wordhunt.py solve

# Dry run (find words but don't swipe)
python wordhunt.py dry-run

# Calibration only
python wordhunt.py calibrate
```

### Testing Without iPhone Mirroring

Use `simulate_board.py` to test word finding on a typed board:

```bash
python simulate_board.py --grid "dtio fors npat lfin" --limit 50
```

## Development

### Creating a Dataset

The dataset creation tool (`create_dataset.py`) allows you to collect labeled letter images from game boards:

1. **Start dataset creation**
   ```bash
   python create_dataset.py --suggest
   ```

2. **Process boards**
   - Ensure iPhone Mirroring is open with Word Hunt game visible
   - The tool captures the current board and extracts individual cell images
   - For each cell, it displays the image and prompts for the letter label
   - Use `--suggest` flag to get AI predictions as starting point (press Enter to accept, or type the correct letter)
   - Type 'q' to quit the current board

3. **Label cells**
   - Each cell image is shown in a window
   - Enter the correct letter (A-Z) for each cell
   - Images are automatically saved to `letter_dataset/<LETTER>/` folders
   - Duplicate images are detected and skipped using content hashing

4. **Continue with more boards**
   - After labeling all 16 cells, you'll be prompted to process another board
   - Type 'y' to continue or 'n' to finish
   - More boards provide better training data diversity

5. **Dataset organization**
   - Images are saved as `letter_dataset/A/A_000001.png`, `letter_dataset/B/B_000001.png`, etc.
   - Each letter folder contains all labeled images for that letter
   - The tool maintains an index to avoid duplicate filenames

### Training Your Own Model

1. **Train model**
   ```bash
   python train_model.py
   ```
   - Automatically splits data into train/validation (80/20)
   - Saves best model to `letter_recognition_model.pth` based on validation accuracy
   - Shows per-letter accuracy diagnostics to identify problematic classes
   - Typical training: 50-100 epochs

2. **Test model**
   - Run solver with new model
   - Check accuracy on real boards
   - Add more training data for problematic letters if needed

### Project Structure

```
WordHunt-Solver/
├── app_cocoa.py              # Cocoa GUI application
├── wordhunt.py               # Core solver logic
├── train_model.py            # CNN training script
├── create_dataset.py         # Dataset creation tool
├── simulate_board.py         # Testing utility
├── letter_recognition_model.pth  # Trained CNN model
├── dictionary.txt            # Word dictionary
├── requirements.txt          # Python dependencies
├── setup.py                  # py2app configuration
├── Makefile                  # Build commands
└── WordHuntSolver.spec       # PyInstaller spec
```

### Key Files Explained

- **`wordhunt.py`**: Main solver class with board detection, OCR, word finding, and automation
- **`app_cocoa.py`**: Native macOS GUI with real-time updates
- **`train_model.py`**: Model training with validation and diagnostics
- **`create_dataset.py`**: Interactive tool for labeling training data
- **`letter_recognition_model.pth`**: Pre-trained PyTorch model weights
- **`dictionary.txt`**: 477,000+ word dictionary for validation

## Technical Details

### Coordinate Systems

The solver handles three coordinate systems:

1. **Window Coordinates**: Top-left origin, relative to iPhone Mirroring window
2. **Screen Coordinates**: Top-left origin, absolute screen position (pyautogui)
3. **Quartz Coordinates**: Bottom-left origin, macOS native (CGEvent)

The solver converts between these systems for accurate mouse control.

### Board Detection Algorithm

1. Crop image to likely board region (45%-85% of height)
2. Convert to HSV color space
3. Create green mask (H: 35-85, S: 40-255, V: 40-255)
4. Apply morphological operations (close + open)
5. Find largest contour (the board)
6. Calculate grid cell positions from bounding box
7. Optionally refine using tan tile detection

### Word Finding Algorithm

1. Build Trie from dictionary (compresses 'qu' → 'q' for QU tiles)
2. For each starting cell:
   - Perform DFS in all 8 directions
   - Track visited cells to avoid reuse
   - Check Trie for valid word completions
3. Sort results by length (descending), then alphabetically
4. Calculate potential score using Word Hunt scoring rules

### Scoring System

- 3 letters: 100 points
- 4 letters: 400 points
- 5 letters: 800 points
- 6 letters: 1,400 points
- 7 letters: 1,800 points
- 8 letters: 2,200 points
- 9+ letters: 2,600 + (length - 9) × 400 points

## Troubleshooting

### Board Not Detected

- Ensure iPhone Mirroring window is visible
- Check that game board is fully visible (not covered)
- Try calibration for better accuracy
- Adjust `WORDHUNT_MARGIN_*_RATIO` environment variables

### Letters Misrecognized

- Run calibration to improve board detection
- Check lighting conditions
- Train model with more data for problematic letters
- Use `--suggest` flag in dataset creation to improve labeling

### Swipes Not Working

- Verify Accessibility permission is granted
- Check that iPhone Mirroring window is focused
- Ensure game is in active play state
- Try "Move to Corner" button to reposition window

### Model Not Found

- Ensure `letter_recognition_model.pth` is in the same directory
- Check file permissions
- Re-download from repository if missing

## Contributing

Contributions are welcome! Areas for improvement:

- Adding to the letter dataset: While the current model achieves high accuracy, additional labeled letter images can help improve edge cases and robustness. See the "Creating a Dataset" section above for instructions on how to add new training data.
- Support for different screen sizes/resolutions
- Performance optimizations
- Additional game modes
- Cross-platform support (Windows/Linux)

### Development Setup

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

**Disclaimer**: This tool is for educational purposes. Use responsibly and in accordance with GamePigeon's terms of service.

**Note**: This README was generated (entirely) with assistance from AI tools. While the technical information is accurate, please verify details against the source code and documentation.
