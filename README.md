# Masai-project-
#include <iostream>
#include <queue>
#include <vector>
#include <algorithm>
#include <fstream>
#include <string>

using namespace std;

struct Job {
    int arrivalTime;
    int coresRequired;
    int memoryRequired; // in GB
    int executionTime;  // in hours
    int id;
    int grossValue() const {
        return coresRequired * memoryRequired * executionTime;
    }
};

struct WorkerNode {
    int id;
    int availableCores = 24;
    int availableMemory = 64; // in GB
};

// Comparator functions for different job scheduling policies
bool compareByArrivalTime(const Job& a, const Job& b) {
    return a.arrivalTime < b.arrivalTime;
}

bool compareBySmallestJob(const Job& a, const Job& b) {
    return a.grossValue() < b.grossValue();
}

bool compareByShortestDuration(const Job& a, const Job& b) {
    return a.executionTime < b.executionTime;
}

class JobScheduler {
private:
    vector<WorkerNode> workerNodes;
    vector<Job> jobQueue; // Use vector instead of queue for better sorting
    int totalJobsProcessed = 0;
    int totalCPUUtilization = 0;
    int totalMemoryUtilization = 0;

public:
    JobScheduler(int numWorkers) {
        for (int i = 0; i < numWorkers; ++i) {
            workerNodes.push_back({i});
        }
    }

    void addJob(const Job& job) {
        jobQueue.push_back(job);
    }

    void scheduleJobs(int queuePolicy, int allocationPolicy) {
        // Sort jobs based on queue policy
        if (queuePolicy == 1) {
            sort(jobQueue.begin(), jobQueue.end(), compareByArrivalTime);
        } else if (queuePolicy == 2) {
            sort(jobQueue.begin(), jobQueue.end(), compareBySmallestJob);
        } else if (queuePolicy == 3) {
            sort(jobQueue.begin(), jobQueue.end(), compareByShortestDuration);
        }

        // Try to allocate jobs based on allocation policy
        for (auto& job : jobQueue) {
            if (allocateJob(job, allocationPolicy)) {
                totalJobsProcessed++;
                totalCPUUtilization += job.coresRequired;
                totalMemoryUtilization += job.memoryRequired;
            }
        }

        // Remove allocated jobs from queue
        jobQueue.erase(remove_if(jobQueue.begin(), jobQueue.end(),
            [](const Job& job) {
                return job.coresRequired == 0;
            }), jobQueue.end());
    }

    bool allocateJob(Job& job, int allocationPolicy) {
        if (allocationPolicy == 1) { // First Fit
            for (auto& worker : workerNodes) {
                if (worker.availableCores >= job.coresRequired &&
                    worker.availableMemory >= job.memoryRequired) {
                    worker.availableCores -= job.coresRequired;
                    worker.availableMemory -= job.memoryRequired;
                    return true;
                }
            }
        } else if (allocationPolicy == 2) { // Best Fit
            auto bestFit = workerNodes.end();
            for (auto it = workerNodes.begin(); it != workerNodes.end(); ++it) {
                if (it->availableCores >= job.coresRequired &&
                    it->availableMemory >= job.memoryRequired) {
                    if (bestFit == workerNodes.end() || 
                        (it->availableCores < bestFit->availableCores && 
                         it->availableMemory < bestFit->availableMemory)) {
                        bestFit = it;
                    }
                }
            }
            if (bestFit != workerNodes.end()) {
                bestFit->availableCores -= job.coresRequired;
                bestFit->availableMemory -= job.memoryRequired;
                return true;
            }
        } else if (allocationPolicy == 3) { // Worst Fit
            auto worstFit = workerNodes.end();
            for (auto it = workerNodes.begin(); it != workerNodes.end(); ++it) {
                if (it->availableCores >= job.coresRequired &&
                    it->availableMemory >= job.memoryRequired) {
                    if (worstFit == workerNodes.end() || 
                        (it->availableCores > worstFit->availableCores && 
                         it->availableMemory > worstFit->availableMemory)) {
                        worstFit = it;
                    }
                }
            }
            if (worstFit != workerNodes.end()) {
                worstFit->availableCores -= job.coresRequired;
                worstFit->availableMemory -= job.memoryRequired;
                return true;
            }
        }
        return false;
    }

    void printStatistics() {
        cout << "Total Jobs Processed: " << totalJobsProcessed << endl;
        cout << "Total CPU Utilization: " << totalCPUUtilization << " Cores" << endl;
        cout << "Total Memory Utilization: " << totalMemoryUtilization << " GB" << endl;
    }

    void generateCSV(const string& filename) {
        ofstream file(filename);
        file << "Jobs Processed,CPU Utilization,Memory Utilization\n";
        file << totalJobsProcessed << "," << totalCPUUtilization << "," << totalMemoryUtilization << "\n";
        file.close();
    }
};

int main() {
    JobScheduler scheduler(128);

    scheduler.addJob({0, 2, 4, 1, 1});
    scheduler.addJob({0, 4, 8, 2, 2});
    scheduler.addJob({0, 1, 2, 1, 3});
    scheduler.addJob({0, 6, 16, 3, 4});

    // Example: FCFS Policy with First Fit Allocation
    scheduler.scheduleJobs(1, 1); // First number for queue policy, second for allocation policy
    scheduler.printStatistics();
    scheduler.generateCSV("scheduler_stats.csv");

    return 0;
}
