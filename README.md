# Data-structure-Pro
The Hospital Patient Monitoring System is a project that manages hospital data using data structures like linked list, queue, stack, tree, graph, and hash table for efficient patient and hospital management.
#include <iostream>
#include <queue>
#include <stack>
#include <unordered_map>
#include <list>
#include <vector>
#include <map>
#include <string>
using namespace std;

struct Patient {
    string id, name, dept;
    int age;
    Patient* next;
    Patient(string pid, string pname, int page, string pdept) {
        id = pid; name = pname; age = page; dept = pdept; next = NULL;
    }
};

class PatientList {
public:
    Patient* head = NULL;
    Patient* tail = NULL;
    void append(Patient* node) {
        if (!head) head = tail = node;
        else { tail->next = node; tail = node; }
    }
    void remove(Patient* node) {
        if (!head) return;
        if (head == node) { head = head->next; return; }
        Patient* cur = head;
        while (cur->next && cur->next != node) cur = cur->next;
        if (cur->next) cur->next = node->next;
    }
    void display() {
        Patient* cur = head;
        while (cur) {
            cout << cur->id << " " << cur->name << " " << cur->age << " " << cur->dept << endl;
            cur = cur->next;
        }
    }
};

struct Appointment {
    string id, pid, doc, time;
    Appointment(string aid, string apid, string adoc, string atime)
        : id(aid), pid(apid), doc(adoc), time(atime) {}
};

class Hospital {
public:
    PatientList plist;
    unordered_map<string, Patient*> table;
    queue<Appointment> aq;
    stack<pair<string, string>> stk;
    map<string, vector<string>> graph;

    void admit(string id, string name, int age, string dept) {
        Patient* p = new Patient(id, name, age, dept);
        plist.append(p);
        table[id] = p;
        stk.push({"admit", id});
    }
    void discharge(string id) {
        if (table.find(id) != table.end()) {
            Patient* p = table[id];
            stk.push({"discharge", id + "," + p->name + "," + to_string(p->age) + "," + p->dept});
            plist.remove(p);
            table.erase(id);
        }
    }
    void undo() {
        if (stk.empty()) return;
        auto act = stk.top(); stk.pop();
        if (act.first == "discharge") {
            string d = act.second;
            size_t pos1 = d.find(",");
            string id = d.substr(0, pos1);
            d.erase(0, pos1 + 1);
            pos1 = d.find(",");
            string name = d.substr(0, pos1);
            d.erase(0, pos1 + 1);
            pos1 = d.find(",");
            int age = stoi(d.substr(0, pos1));
            d.erase(0, pos1 + 1);
            string dept = d;
            admit(id, name, age, dept);
        } else if (act.first == "admit") {
            string id = act.second;
            if (table.find(id) != table.end()) {
                Patient* p = table[id];
                plist.remove(p);
                table.erase(id);
            }
        }
    }
    void addEdge(string a, string b) {
        graph[a].push_back(b);
        graph[b].push_back(a);
    }
    vector<string> bfs(string start, string end) {
        queue<string> q;
        map<string, string> prev;
        q.push(start);
        prev[start] = "";
        while (!q.empty()) {
            string cur = q.front(); q.pop();
            if (cur == end) {
                vector<string> path;
                while (cur != "") {
                    path.push_back(cur);
                    cur = prev[cur];
                }
                reverse(path.begin(), path.end());
                return path;
            }
            for (auto nb : graph[cur]) {
                if (prev.find(nb) == prev.end()) {
                    prev[nb] = cur;
                    q.push(nb);
                }
            }
        }
        return {};
    }
};

int main() {
    Hospital h;
    h.admit("P1", "Arun", 30, "Cardiology");
    h.admit("P2", "Priya", 25, "Neurology");
    cout << "Patients List:\n";
    h.plist.display();
    h.discharge("P1");
    cout << "\nAfter Discharge:\n";
    h.plist.display();
    h.undo();
    cout << "\nAfter Undo:\n";
    h.plist.display();
    h.addEdge("Ward", "Lab");
    h.addEdge("Lab", "Pharmacy");
    vector<string> path = h.bfs("Ward", "Pharmacy");
    cout << "\nPath from Ward to Pharmacy: ";
    for (auto x : path) cout << x << " ";
    cout << endl;
    return 0;
}
