
Design Dropbox.


Functional Requirements:

    1. Users should be able to upload and download their files/photos from any device.
    2. Users should be able to share files or folders with other users.
    3. Our service should support automatic synchronization between devices, i.e., after updating a file on one device, it should get synchronized on all devices.
    4. The system should support storing large files up to a GB.
    5. Our system should support offline editing. Users should be able to add/delete/modify files while offline, and as soon as they come online, all their changes should be synced to the remote servers and other online devices.
    6. The system should support snapshotting of the data, so that users can go back to any version of the files.

Non-functional Requirements:

    ACID-ity is required and guaranteed.

Design Considerations:

    Expect huge read and write volumes.
    Read-to-write ratio is nearly the same.
    Files can be stored in small parts or chunks (say 5MB), in the case of a failed uploaded, the next attempt should start from the last failed chunk.
    We can reduce the number of operations by transferring only the updated chunks.
    Removing duplicate chunks can save storage space and bandwidth usage.
    We can keep a local copy of metadata for uploaded files etc.

Capacity Estimation and Constraints:

    1. Assume 500 million total users, and 100 million daily active users (DAU).

    2. Assume on average, each user connects from three different devices.

    3. Assume, on average, each user has 200 files/photos, there could be 100 at least 1 billion files total.

    4. Assume average file size is 100kb.

    5. Assume there will be 1 million active connections per minute.

    If the average file size is 100kb, then 100kb * 1 billion total files = 10 PB.

    1 million active connections per minute = 20000 per second on average.

    If 3 devices on average are connected and there are 500 million users, then there could be 15 million devices connected at any time.

Services:

    Synchronization
    Message Queueing

Database Modeling:

    Metadata:
        Tables
            - Chunks
            - Files
            - Users
            - Devices
            - Workspace (folders)










