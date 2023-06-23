#EXP 24: EMPLOYEE ONBOARDING APPLICATION
## AIM:
To create an empolyee onboarding application using React,SpringBoot and SQL.
## ALGORITHM:
1. Create a SpringBoot project with the required specifications.
2. Set up the JDK environment and add the necessary java files.
3. Create a suitable database in PostgresSQL and connect it to your SpringBoot File.
4. Create the User Interface using React and define the specific components.
5. Connect the front end to the back end.
6. Run the project and make necessary changes as required.
## PROGRAM:
## SPRING BOOT:
## Employee.java:
```
package com.saveetha.employee.emp;
import jakarta.persistence.*;
import java.time.LocalDate;
import java.time.Period;
@Entity
@Table
public class Employee {
   @Id
   @SequenceGenerator(
           name = "employee_sequence",
           sequenceName = "employee_sequence",
           allocationSize = 1
   )
   @GeneratedValue(
           strategy = GenerationType.SEQUENCE,
           generator = "employee_sequence"
   )
   private long employeeID;
   private String employeeName;
   @Transient
   private Integer employeeAge;
   private LocalDate employeeDOB;
   private String employeeEmail;
   public Employee() {
   }
   public Employee(long employeeID, String employeeName,  LocalDate employeeDOB, String employeeEmail) {
       this.employeeID = employeeID;
       this.employeeName = employeeName;
       this.employeeDOB = employeeDOB;
       this.employeeEmail = employeeEmail;
   }
   public long getEmployeeID() {return employeeID;}
   public void setEmployeeID(long employeeID) {this.employeeID = employeeID;}
   public String getEmployeeName() {return employeeName;}
   public void setEmployeeName(String employeeName) {this.employeeName = employeeName;}
   public Integer getEmployeeAge() {return Period.between(employeeDOB,LocalDate.now()).getYears();}
   public void setEmployeeAge(Integer employeeAge) {this.employeeAge = employeeAge;}
   public LocalDate getEmployeeDOB() {return employeeDOB;}
   public void setEmployeeDOB(LocalDate employeeDOB) {this.employeeDOB = employeeDOB;}
   public String getEmployeeEmail() {return employeeEmail;}
   public void setEmployeeEmail(String employeeEmail) {
       this.employeeEmail = employeeEmail;
   }
   @Override
   public String toString() {
       return "Employee{" +
               "employeeID=" + employeeID +
               ", employeeName='" + employeeName + '\'' +
               ", employeeAge=" + employeeAge +
               ", employeeDOB=" + employeeDOB +
               ", employeeEmail='" + employeeEmail + '\'' +
               '}';
   }
}
```
### EmployeeController.java
```
package com.saveetha.employee.emp;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import java.util.List;
@RestController
@RequestMapping(path = "/api/v1/employee")
public class EmployeeController {
    private final EmployeeService employeeService;
    @Autowired
    public EmployeeController(EmployeeService employeeService) {
        this.employeeService = employeeService;
    }
    @GetMapping("/")
    public List<Employee> getEmployeeDetails()
    {
        return employeeService.displayEmployeeDetails();
    }
    @PostMapping("/")
    public void postEmployee(@RequestBody Employee employee){
        employeeService.registerNewEmployee(employee);
    }
    @DeleteMapping(path = {"/{id}"})
    public void deleteEmployee(@PathVariable("id") Long employeeId){
        employeeService.removeEmployee(employeeId);
    }
}
```
### REACT CODES:
### App.js:
```
import React from 'react';
import './App.css';
import { BrowserRouter as Router, Route, Routes, Link } from "react-router-dom";
import EmployeeDirectoryComponent from './components/EmployeeDirectoryComponent/EmployeeDirectoryComponent';
import EmployeeRegistrationComponent from './components/EmployeeRegistrationComponent/EmployeeRegistrationComponent';
import EmployeeDeletionComponent from './components/EmployeeDeletionComponent/EmployeeDeletionComponent';
import HeaderComponent from './components/HeaderComponent/HeaderComponent';

function App() {
  return (
    <Router>
      <div className='container'>
        <center>
        <HeaderComponent/>
        </center>
      
      <nav className='nav-menu'>
        <Link to='/'>Employee List</Link>
        <Link to='/admin/add'>Add Employee</Link>
        <Link to='/admin/delete'>Delete Employee</Link>
      </nav>
      <Routes>
        <Route exact path = '/' element={<EmployeeDirectoryComponent/>}></Route>
        <Route path = '/admin/add' element={<EmployeeRegistrationComponent/>}></Route>
        <Route path = '/admin/delete' element={<EmployeeDeletionComponent/>}></Route>
      </Routes>
      </div>
    </Router>
  );
}

export default App;
```
### EmployeeDirectoryComponent.js:
```
import React, { useEffect, useState } from 'react';
import './EmployeeDirectoryComponent.css'; 

function EmployeeDirectoryComponent() {
  const [employees, setEmployees] = useState([]);

  useEffect(() => {
    fetchEmployees();
  }, []);

  const fetchEmployees = async () => {
    try {
      const response = await fetch('http://localhost:8080/api/v1/employee/');
      const data = await response.json();
      setEmployees(data);
    } catch (error) {
      console.error('Error:', error);
    }
  };

  return (
    <div className="employee-list">
      {employees.map((employee) => (
        <div className="employee-card" key={employee.employeeID}>
          <p>Employee ID: {employee.employeeID}</p>
          <p>Name: {employee.employeeName}</p>
          <p>Age: {employee.employeeAge}</p>
          <p>Date of Birth: {employee.employeeDOB}</p>
          <p>Email: {employee.employeeEmail}</p>
        </div>
      ))}
    </div>
  );
};

export default EmployeeDirectoryComponent;
```
### EmployeeRegistrationComponent.js:
```
import React, { useState } from "react";
import './EmployeeRegistrationComponent.css'

function EmployeeRegistrationComponent() {
    const [employee, setEmployees] = useState({
        employeeName: '',
        employeeDOB: '',
        employeeEmail: ''
    });

    const handleChange = (event) => {
        const {name, value} = event.target;
        setEmployees({...employee, [name]:value});
    };

    const handleFormSubmit =async (event) => {
        event.preventDefault();
        await fetch('http://localhost:8080/api/v1/employee/', {
            method: 'POST',
            headers: {
                'Content-type': 'application/json'
            },
            body: JSON.stringify(employee)
        })

        .then((response) => {
            if (response.status == 500)
            {
                alert(`Error!`)
            }
            else{
                alert(`Data of ${employee.employeeName} is added successfully`)
                window.location.href = '/'
            }
        })
  };
            
    return(
        <form onSubmit={handleFormSubmit}>
            <label>
                Employee Name: 
                <input type="text" name="employeeName" value={employee.employeeName} onChange={handleChange} required/>
            </label> 
            <label>
                Employee DOB: 
                <input type="date" name="employeeDOB" value={employee.employeeDOB} onChange={handleChange} required/>
            </label> 
            <label>
                Employee Email: 
                <input type="email" name="employeeEmail" value={employee.employeeEmail} onChange={handleChange} required/>
            </label>

            <button type="submit">SUBMIT</button> 
        </form>
    )
}

export default EmployeeRegistrationComponent;
```
### EmployeeDeletionComponent.js:
```
import React, { useState } from 'react';
import './EmployeeDeletionComponent.css'
function EmployeeDeletionComponent() {
  const [employeeID, setEmployeeID] = useState('');

  const handleChange = (event) => {
    setEmployeeID(event.target.value);
  };

  const handleSubmit = async(event) => {
    event.preventDefault();
    await fetch(`http://localhost:8080/api/v1/employee/${employeeID}`,{
      method: 'DELETE'
    })
    .then((response) => {
            if (response.status == 500)
            {
                alert(`Error!`)
            }
            else{
                alert(`Data deleted successfully`)
                window.location.href = '/'
            }
        })
  };

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Employee ID:
        <input
          type="number"
          name="employeeID"
          value={employeeID}
          onChange={handleChange}
          required
        />
      </label>
      <button type="submit">Submit</button>
    </form>
  );
};

export default EmployeeDeletionComponent;
```
## OUTPUT:
### Home Page (Get Employee Details):
![247887144-26a6bc91-5584-4c7e-a963-67f2c7e11255](https://github.com/harshavardhini33/Employee-Onboarding-Application/assets/93427208/04deaca9-ba6b-4449-9bfd-0c71ae74e59b)
### Add Employee Details:
![247887441-de689e57-01ef-4c89-9fdc-f8066eab7407](https://github.com/harshavardhini33/Employee-Onboarding-Application/assets/93427208/f2840af5-1eb7-4f3b-8985-ac3d780c4fc0)
![247887503-46a18c54-a871-4ee2-b62f-360f606fc1a9](https://github.com/harshavardhini33/Employee-Onboarding-Application/assets/93427208/4718bbe9-75ea-4662-83c9-35f22e442245)


### Delete Employee Details:
![247887679-d165d061-05fd-47cc-becc-7fa3e0d41bd3](https://github.com/harshavardhini33/Employee-Onboarding-Application/assets/93427208/b4061cbf-a150-4525-9fd2-68124d5b1888)

![247887753-8865a005-68ca-4e50-a5ea-f74425e3ac59](https://github.com/harshavardhini33/Employee-Onboarding-Application/assets/93427208/1e74fcc8-acee-4fd5-bdb0-eaff187872a8)

## RESULT:
Thus, an employee onboarding app is created using React, springboot and SQL.
