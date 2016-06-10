stride算法实现:

class Simulator:
    def __init__(self):
        self.job_list = []
        self.time_slice = 10

    def add_job(self, job_name, job_runtime, job_prio):
        print("Adding process " + str(job_name) + " with priority " + str(job_prio) + " needing " +
              str(job_runtime) + " to run")
        self.job_list.append({'name': job_name, 'runtime': job_runtime, 'step': 1.0 / job_prio, 'stride': 0.0,
                              'job_finished_time': 0})

    def run(self):
        while True:
            if len(self.job_list) == 0:
                return
            next_run = 0
            for item in range(1, len(self.job_list)):
                if self.job_list[item]['stride'] < self.job_list[next_run]['stride']:
                    next_run = item
            cur_proc = self.job_list[next_run]
            run_time = cur_proc['runtime'] - cur_proc['job_finished_time']
            cur_proc['stride'] += cur_proc['step']
            if run_time > self.time_slice:
                run_time = self.time_slice
            print("Running process " + str(cur_proc['name']) + " for " + str(run_time) + "ms")
            cur_proc['job_finished_time'] += run_time
            if cur_proc['job_finished_time'] >= cur_proc['runtime']:
                print("Process " + str(cur_proc['name']) + " finished")
                del self.job_list[next_run]



if __name__ == '__main__':
    sim = Simulator()
    sim.add_job('task1', 60, 2)
    sim.add_job('task2', 100, 4)
    sim.add_job('task3', 150, 1)
    sim.add_job('task4', 50, 6)
    sim.run()
