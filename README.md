# Youtube-Video-to-Frame


## Capture All Frame from Playlist
```python
from pytube import Playlist, YouTube
import cv2
import os

def download_video(url, output_path):
    yt = YouTube(url)
    ys = yt.streams.filter(file_extension='mp4', res='720p').first()
    ys.download(output_path)
    return ys.default_filename

def capture_frames(video_path, output_folder, interval=2):
    cap = cv2.VideoCapture(video_path)
    fps = int(cap.get(cv2.CAP_PROP_FPS))
    frame_count = 0

    while cap.isOpened():
        ret, frame = cap.read()

        if not ret:
            break

        if frame_count % (fps * interval) == 0:
            frame_filename = f"{output_folder}/frame_{frame_count}.png"
            cv2.imwrite(frame_filename, frame)

        frame_count += 1

    cap.release()
    cv2.destroyAllWindows()

def process_playlist(playlist_url, output_folder):
    playlist = Playlist(playlist_url)

    for index, video_url in enumerate(playlist.video_urls):
        print(f"Processing video {index + 1}/{len(playlist)}: {video_url}")
        
        try:
            video_filename = download_video(video_url, output_folder)
            video_path = os.path.join(output_folder, video_filename)
            
            frames_output_folder = f"{output_folder}/frames_video_{index + 1}"
            os.makedirs(frames_output_folder, exist_ok=True)
            
            capture_frames(video_path, frames_output_folder)
            
            print(f"Frames for video {index + 1} captured successfully.")
        except Exception as e:
            print(f"Error processing video {index + 1}: {str(e)}")

if __name__ == "__main__":
    playlist_url = "Youtube Playlist Link"
    output_folder = "OUTPUT_FOLDER"
    os.makedirs(output_folder, exist_ok=True)

    process_playlist(playlist_url, output_folder)

    print("All videos in the playlist processed successfully.")

```

## Remove Duplicates From Frames

```python
from PIL import Image
import imagehash
import os
from pathlib import Path
from tqdm import tqdm

def find_duplicate_images(input_folder, output_folder):
    input_folder = Path(input_folder)
    output_folder = Path(output_folder)

    # Create the output folder if it doesn't exist
    output_folder.mkdir(parents=True, exist_ok=True)

    hash_dict = {}

    # Iterate through all images in the input folder
    for image_file in tqdm(input_folder.glob("*.png")):
        with Image.open(image_file) as img:
            # Calculate the hash of the image
            img_hash = str(imagehash.average_hash(img))

            # Check if the hash is already in the dictionary
            if img_hash not in hash_dict:
                # If not, save the image to the output folder
                hash_dict[img_hash] = image_file
                img.save(output_folder / image_file.name)

if __name__ == "__main__":
    input_folder = 'OUTPUT_FOLDER/frames_video_2/'
    output_folder = "Duplicates/Frame2"  # Replace with the desired output folder path

    find_duplicate_images(input_folder, output_folder)

    print("Duplicate images removed successfully.")
```
