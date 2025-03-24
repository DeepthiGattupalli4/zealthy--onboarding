// Root project structure: 
// - backend (Spring Boot) 
// - frontend (React) 
// PART 1: BACKEND (Spring Boot with Java 17) 
// Path: backend/src/main/java/com/zealthy/onboarding 
// User.java (Entity) 
@Entity 
public class User { 
@Id 
@GeneratedValue(strategy = GenerationType.IDENTITY) 
private Long id; 
private String email; 
private String password; 
private String aboutMe; 
private String birthdate; 
private String street; 
private String city; 
private String state; 
private String zip; 
} 
// AdminConfig.java (Entity) 
@Entity 
public class AdminConfig { 
@Id 
@GeneratedValue(strategy = GenerationType.IDENTITY) 
private Long id; 
private int pageNumber; // 2 or 3 
@ElementCollection 
private List<String> components; // e.g., ABOUT_ME, ADDRESS, BIRTHDATE 
} 
// UserRepository.java 
public interface UserRepository extends JpaRepository<User, Long> {} 
// AdminConfigRepository.java 
public interface AdminConfigRepository extends JpaRepository<AdminConfig, Long> { 
Optional<AdminConfig> findByPageNumber(int pageNumber); 
} 
// OnboardingController.java 
@RestController 
@RequestMapping("/api") 
public class OnboardingController { 
@Autowired 
    private UserRepository userRepository; 
  
    @Autowired 
    private AdminConfigRepository configRepository; 
  
    @PostMapping("/register") 
    public User register(@RequestBody User user) { 
        return userRepository.save(user); 
    } 
  
    @PostMapping("/onboarding/page/{pageNum}") 
    public User updatePage(@PathVariable int pageNum, @RequestBody User userData) { 
        User user = userRepository.findById(userData.getId()).orElseThrow(); 
        // Update fields based on pageNum logic (simplified) 
        if (pageNum == 2) { 
            user.setAboutMe(userData.getAboutMe()); 
            user.setBirthdate(userData.getBirthdate()); 
        } else if (pageNum == 3) { 
            user.setStreet(userData.getStreet()); 
            user.setCity(userData.getCity()); 
            user.setState(userData.getState()); 
            user.setZip(userData.getZip()); 
        } 
        return userRepository.save(user); 
    } 
  
@GetMapping("/config") 
public List<AdminConfig> getConfig() { 
return configRepository.findAll(); 
} 
@PostMapping("/config") 
public List<AdminConfig> setConfig(@RequestBody List<AdminConfig> configs) { 
return configRepository.saveAll(configs); 
} 
@GetMapping("/users") 
public List<User> getAllUsers() { 
return userRepository.findAll(); 
} 
} 
// application.properties 
spring.datasource.url=jdbc:postgresql://localhost:5432/zealthy_db 
spring.datasource.username=postgres 
spring.datasource.password=postgres 
spring.jpa.hibernate.ddl-auto=update 
spring.jpa.show-sql=true 
Front end 
Follow the steps below for the front-end development. 
// PART 2: FRONTEND (React + Vite) 
// Run in terminal: 
// npm create vite@latest frontend -- --template react 
// cd frontend && npm install axios react-router-dom 
// main.jsx 
import React from 'react'; 
import ReactDOM from 'react-dom/client'; 
import App from './App'; 
import { BrowserRouter } from 'react-router-dom'; 
ReactDOM.createRoot(document.getElementById('root')).render( 
<BrowserRouter> 
<App /> 
</BrowserRouter> 
); 
// App.jsx 
import { Routes, Route } from 'react-router-dom'; 
import OnboardingWizard from './pages/OnboardingWizard'; 
import Admin from './pages/Admin'; 
import DataTable from './pages/DataTable'; 
export default function App() { 
return ( 
<Routes> 
<Route path="/" element={<OnboardingWizard />} /> 
<Route path="/admin" element={<Admin />} /> 
<Route path="/data" element={<DataTable />} /> 
</Routes> 
); 
} 
// pages/OnboardingWizard.jsx 
import React, { useState, useEffect } from 'react'; 
import axios from 'axios'; 
export default function OnboardingWizard() { 
  const [step, setStep] = useState(1); 
  const [form, setForm] = useState({}); 
  const [components, setComponents] = useState({2: [], 3: []}); 
  
  useEffect(() => { 
    axios.get('/api/config').then(res => { 
      const cfg = {2: [], 3: []}; 
      res.data.forEach(conf => cfg[conf.pageNumber] = conf.components); 
      setComponents(cfg); 
    }); 
  }, []); 
  
  const handleNext = () => { 
    if (step === 1) axios.post('/api/register', form).then(res => { 
      setForm(res.data); 
      setStep(2); 
    }); 
    else if (step === 2 || step === 3) axios.post(`/api/onboarding/page/${step}`, 
form).then(res => { 
      setForm(res.data); 
      setStep(step + 1); 
    }); 
  }; 
  
  const renderFields = () => { 
    if (step === 1) return ( 
      <> 
        <input placeholder="Email" onChange={e => setForm({...form, email: 
e.target.value})} /> 
        <input placeholder="Password" type="password" onChange={e => 
setForm({...form, password: e.target.value})} /> 
      </> 
    ); 
  
    return components[step]?.map(c => { 
      if (c === 'ABOUT_ME') return <textarea placeholder="About Me" 
onChange={e => setForm({...form, aboutMe: e.target.value})} />; 
      if (c === 'BIRTHDATE') return <input type="date" onChange={e => 
setForm({...form, birthdate: e.target.value})} />; 
      if (c === 'ADDRESS') return ( 
        <> 
          <input placeholder="Street" onChange={e => setForm({...form, street: 
e.target.value})} /> 
          <input placeholder="City" onChange={e => setForm({...form, city: 
e.target.value})} /> 
          <input placeholder="State" onChange={e => setForm({...form, state: 
e.target.value})} /> 
          <input placeholder="Zip" onChange={e => setForm({...form, zip: 
e.target.value})} /> 
        </> 
      ); 
    }); 
  }; 
  
  return ( 
    <div> 
      <h2>Step {step} of 3</h2> 
      {renderFields()} 
      {step <= 3 && <button onClick={handleNext}>Next</button>} 
    </div> 
  ); 
} 
  
// pages/Admin.jsx 
import React, { useState, useEffect } from 'react'; 
import axios from 'axios'; 
  
const COMPONENTS = ['ABOUT_ME', 'ADDRESS', 'BIRTHDATE']; 
  
export default function Admin() { 
  const [config, setConfig] = useState({2: [], 3: []}); 
  
  useEffect(() => { 
    axios.get('/api/config').then(res => { 
      const cfg = {2: [], 3: []}; 
      res.data.forEach(c => cfg[c.pageNumber] = c.components); 
      setConfig(cfg); 
    }); 
  }, []); 
  
  const toggle = (page, comp) => { 
    const list = config[page]; 
    setConfig({...config, [page]: list.includes(comp) ? list.filter(c => c !== 
comp) : [...list, comp]}); 
  }; 
  
  const save = () => { 
    const payload = [2, 3].map(p => ({pageNumber: p, components: config[p]})); 
    axios.post('/api/config', payload); 
  }; 
  
  return ( 
    <div> 
      {[2,3].map(p => ( 
        <div key={p}> 
          <h4>Page {p}</h4> 
          {COMPONENTS.map(c => ( 
            <label key={c}> 
              <input type="checkbox" checked={config[p].includes(c)} onChange={() 
=> toggle(p, c)} /> {c} 
            </label> 
          ))} 
        </div> 
      ))} 
      <button onClick={save}>Save</button> 
    </div> 
  ); 
} 
  
// pages/DataTable.jsx 
import React, { useEffect, useState } from 'react'; 
import axios from 'axios'; 
  
export default function DataTable() { 
  const [users, setUsers] = useState([]); 
  
  useEffect(() => { 
    axios.get('/api/users').then(res => setUsers(res.data)); 
  }, []); 
  
  return ( 
    <table border="1"> 
      <thead> 
        <tr> 
          <th>Email</th><th>About</th><th>Birthdate</th><th>Address</th> 
        </tr> 
      </thead> 
      <tbody> 
        {users.map(u => ( 
          <tr key={u.id}> 
            <td>{u.email}</td> 
            <td>{u.aboutMe}</td> 
            <td>{u.birthdate}</td> 
            <td>{u.street}, {u.city}, {u.state} - {u.zip}</td> 
          </tr> 
        ))} 
      </tbody> 
    </table> 
  ); 
} 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
Project structure looks like below 
 
If you want to run the backend Spring boot code, do it like below 
 
If you want to run UI application follow below commands to make it up. 
Access the app at: http://localhost:5173 
Backend endpoint: 
We can also run this using Docker images with below steps 
To Run Locally with Docker: 
1. Put these files in your root project directory (zealthy-project/)  
2. From the terminal, run: 
docker-compose up –build 
3. Access: 
• Frontend: http://localhost:5173 
• Backend API: http://localhost:8080/api 
• DB: PostgreSQL at localhost:5432 
