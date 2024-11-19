# Code_Generation_LLM
## 1. Requirement:
- numpy == 1.26.4
- sklearn == 1.5.1
- pytorch == 2.3.0
- opencv == 4.10.0
- easyocr == 1.7.2
- openai == 1.37.0

## 2. Framework:
![Image](https://raw.githubusercontent.com/Golden-Arc/Code_Generation_LLM/main/img/llm.svg)

## 3. Module details：
### 3.1 Mouse Pointer Detction:
- Pointer detection involves locating the mouse pointer position on image frames derived from the original video, divided into three steps:
  - Step 1: Initial Screening of Change Areas via Frame Differencing: This step uses frame differencing based on full-image comparison to preliminarily screen areas of change. A single change region between consecutive frames, with a size range of (110,400) pixels, is selected.
  - Step 2: Single Frame Clustering (K-means): The K-means algorithm clusters all detected change regions on each frame, using the points that meet the criteria from Step 1. The number of cluster centers is set to 1.
  - Step 3: Sequential Clustering (DBSCAN): The DBSCAN algorithm clusters the entire sequence of changed frames, considering groups of three frames from before and after (i.e., non-consecutive frames in time sequence). The clustering distance is set to 20, with the largest resulting cluster selected as the central point of clustering.
### 3.2 Global Frame Differencial:
- The goal of global frame differencial is to detect keyframes with significant content changes across the entire page, designated as global keyframes.
- Constraints:
  - Frames with differences greater than 0.2 are selected.
  - The time interval between keyframes is at least 1/5 second (equal to 12 frames).
### 3.3 Local Frame Differencial:
- The goal of local frame Differencial is to identify keyframes with significant color differences, designated as local keyframes.
- Constraints:
  - A 40x40 area around the pointer is extracted as the comparison basis in frames that meet near-static requirements (based on pointer detection results, frames where pointer movement is under 3 pixels are considered near-static).
  - To reduce the impact of pointer contour differences, black-and-white pixels are excluded (determined by is_color_pixel to confirm if pixels are colored).
  - Frames with differences greater than 10 are selected.
### 3.4 Keyframes Aggregation:
- Aggregation criteria: All global keyframes are included in the final keyframe list. Local keyframes are also included if there is no global keyframe within 5 frames of their position.
### 3.5 OCR：
- The purpose of OCR is to extract text information from all keyframes.
- The base model is easyocr, which uses the CRAFT algorithm for detection and CRNN for recognition. The CRNN model comprises three components: feature extraction with ResNet, sequence labeling with LSTM, and decoding with CTC. This deep learning process is implemented in PyTorch.
### 3.6 Prompt Constructing:
- In prompt construction, the aim is for the large model to understand task objectives and integrate available information to infer and generate complete, accurate control code.
- Prerequisite Tasks: Equip the model with knowledge to comprehend information related to mouse pointer, keyframes, and the experimental platform, and align it accordingly.
- Core Tasks: Utilize the model’s inferencing capability to deduce necessary operational steps under various experimental conditions and generate control code.
- Prompt Structure: 
  - Background introduction
  - Prerequisite Task 1 (understanding keyframe and pointer concepts and generating code framework)
  - Prerequisite Task 2 (understanding text information in keyframe images and aligning it with workflow)
  - Core Task (model inference to achieve specific experimental objectives)
  - Result output (refining the code framework to produce control code needed for the experimental objective).
### 3.7 Generate Object Code by using LLM:
- Model selection: gpt-4o
- We aim to employ a chain-of-thought framework for progressive task breakdown, enhancing output robustness.
- The prompts from Section 3.6 is divided into 5 sub-prompts.
- After each sub-prompt is generated, the model’s response is combined with the next sub-prompt as input for the following generation step.

## 4. Result：
- The code segment is the direct output from GPT-4o. The large model accurately understood the task objectives and generated correct, ready-to-use object code. The use of chain-of-thought also ensured stable output even after multiple rounds of experiment.
```python
import pyautogui
import time

# Dictionary of frame numbers and corresponding coordinates
coordinates = {
    98: (1011, 76),
    132: (1012, 74),
    230: (1049, 747),
    273: (1430, 211),
    341: (1400, 366),
    371: (1221, 684),
    577: (1809, 176),  # 'Apply' click
    645: (1834, 982),
    690: (1833, 991),
    734: (1832, 996),
    859: (1020, 862),
    911: (1046, 664),
    973: (1052, 658),
    1000: (1048, 652),
    1015: (1049, 653)
}

# Coordinates for the missing steps checkboxes
missing_steps_coordinates = {
    'B': (801 + 390, 725),  # Add discrete amounts of B
    'C': (793 + 390, 845),  # Add discrete amounts of C
    'D': (793 + 390, 905)   # Add discrete amounts of D
}

# Function to simulate mouse clicks
def simulate_clicks(coordinates):
    for frame, (x, y) in coordinates.items():
        if frame == 577:
            # Insert missing steps before clicking 'Apply'
            for step, (step_x, step_y) in missing_steps_coordinates.items():
                pyautogui.moveTo(step_x, step_y, duration=0.5)
                pyautogui.click()
                time.sleep(0.5)
        
        # Move the mouse to the specified coordinates
        pyautogui.moveTo(x, y, duration=0.5)
        # Click at the current mouse position
        pyautogui.click()
        # Optional: Wait for a short period before the next action
        time.sleep(0.5)

# Run the simulation
simulate_clicks(coordinates)
```
- A 3D figure showing how the code generation is affected by the LLM’s parameter

![Image](https://raw.githubusercontent.com/Golden-Arc/Code_Generation_LLM/main/img/chart.png)

## 6. Ablation Experiment:
<style type="text/css">
.tg  {border-collapse:collapse;border-spacing:0;}
.tg td{border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px;
  overflow:hidden;padding:10px 5px;word-break:normal;}
.tg th{border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px;
  font-weight:normal;overflow:hidden;padding:10px 5px;word-break:normal;}
.tg .tg-0pky{border-color:inherit;text-align:left;vertical-align:top}
</style>
<table class="tg"><thead>
  <tr>
    <th class="tg-0pky">Model</th>
    <th class="tg-0pky">Accuracy</th>
  </tr></thead>
<tbody>
  <tr>
    <td class="tg-0pky" colspan="2"><span style="font-weight:bold">Keyframe Detection</span></td>
  </tr>
  <tr>
    <td class="tg-0pky">Global Contour</td>
    <td class="tg-0pky">90%</td>
  </tr>
  <tr>
    <td class="tg-0pky">&nbsp;&nbsp;+ Local Color</td>
    <td class="tg-0pky">100%</td>
  </tr>
  <tr>
    <td class="tg-0pky" colspan="2"><span style="font-weight:bold">Pointer Detection</span></td>
  </tr>
  <tr>
    <td class="tg-0pky">Frame Differential Screening</td>
    <td class="tg-0pky">28.58%</td>
  </tr>
  <tr>
    <td class="tg-0pky">&nbsp;&nbsp;+ Kmeans</td>
    <td class="tg-0pky">35.71%</td>
  </tr>
  <tr>
    <td class="tg-0pky">&nbsp;&nbsp;+ DBSCAN</td>
    <td class="tg-0pky">85.71%</td>
  </tr>
  <tr>
    <td class="tg-0pky">Full Model (Screening + Kmeans + DBSCAN)</td>
    <td class="tg-0pky">100%</td>
  </tr>
</tbody>
</table>