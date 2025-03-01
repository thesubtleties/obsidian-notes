Navigation.css
:root {

  --primary-color: #3a6491;

  --primary-light: #d4e3f7;

  --secondary-color: #f0d6e8;

  --accent-color: #66b088;

  --text-primary: #1a2634;

  --text-secondary: #405165;

  --background-light: #f0f4f9;

  --background-white: #ffffff;

  --border-color: #dde5f0;

  --hover-color: #edf2f9;

  --error-color: #e57373;

  --gradient-primary: linear-gradient(135deg, #3a6491, #4a7bb7);

  --gradient-accent: linear-gradient(135deg, #66b088, #7fc2a0);

}

  

.profile-dropdown {

  position: absolute;

  top: 100%;

  right: 0;

  margin-top: 12px;

  background-color: var(--background-white);

  border-radius: 12px;

  box-shadow: 0 4px 20px rgba(58, 100, 145, 0.15);

  width: 280px;

  overflow: hidden;

  z-index: 1000;

  border: 1px solid rgba(58, 100, 145, 0.12);

}

  

.navbar {

  position: fixed;

  top: 0;

  left: 0;

  right: 0;

  height: 70px;

  background: linear-gradient(to right, #3a6491, #4875a5);

  border-bottom: 1px solid rgba(255, 255, 255, 0.1);

  z-index: 1000;

  box-shadow: 0 2px 12px rgba(58, 100, 145, 0.15);

}

  

.nav-content {

  display: flex;

  justify-content: space-between;

  align-items: center;

  height: 100%;

  padding: 0 24px;

  max-width: 1600px;

  margin: 0 auto;

}

  

.nav-left {

  margin-left: 0;

  display: flex;

  align-items: center;

}

  

.nav-right {

  display: flex;

  align-items: center;

  gap: 20px;

}

  

.logo {

  text-decoration: none;

  padding: 8px 0;

  display: flex;

  align-items: center;

}

  

.logo-text {

  background: linear-gradient(45deg, #ffffff, #e1eaf6);

  -webkit-background-clip: text;

  -webkit-text-fill-color: transparent;

  font-size: 1.8rem;

  font-weight: 700;

  letter-spacing: -0.5px;

  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);

  margin-right: 8px;

  position: relative;

}

  

.logo-text::after {

  content: '';

  position: absolute;

  bottom: -4px;

  left: 0;

  width: 100%;

  height: 2px;

  background: linear-gradient(

    90deg,

    rgba(255, 255, 255, 0.8),

    rgba(255, 255, 255, 0)

  );

  border-radius: 2px;

}

  

.nav-button {

  text-decoration: none;

  color: #ffffff;

  padding: 8px 16px;

  border-radius: 6px;

  transition: all 0.2s ease;

  background: rgba(255, 255, 255, 0.1);

}

  

.nav-button:hover {

  background: rgba(255, 255, 255, 0.2);

  transform: translateY(-1px);

}

  

button.nav-button {

  background: none;

  border: none;

  cursor: pointer;

  font-size: 1rem;

}

  

/* Add margin to the main content to account for fixed navbar */

main {

  margin-top: 4rem;

}

  

/* Update SideNav top position to match new navbar height */

.sideNav {

  top: 70px;

  height: calc(100vh - 70px);

}

  

.profile-button-container {

  position: relative;

}

  

.profile-button {

  background: linear-gradient(135deg, #66b088, #7fc2a0);

  border: none;

  padding: 10px;

  cursor: pointer;

  border-radius: 50%;

  width: 48px;

  height: 48px;

  display: flex;

  align-items: center;

  justify-content: center;

  transition: all 0.2s ease;

  box-shadow: 0 4px 12px rgba(102, 176, 136, 0.2);

}

  

.profile-button:hover {

  background: linear-gradient(135deg, #7fc2a0, #66b088);

  transform: translateY(-2px);

  box-shadow: 0 6px 16px rgba(102, 176, 136, 0.25);

}

  

.profile-button i {

  font-size: 24px;

  color: white;

}

  

.profile-info {

  padding: 24px;

  display: flex;

  align-items: flex-start;

  gap: 16px;

  background: linear-gradient(135deg, #3a6491, #4875a5);

  color: white;

}

  

.profile-icon {

  font-size: 38px;

  color: #ffffff;

}

  

.user-details {

  display: flex;

  flex-direction: column;

  gap: 4px;

}

  

.welcome {

  font-weight: 600;

  font-size: 1.1rem;

  color: #ffffff;

}

  

.email {

  font-size: 0.9rem;

  color: rgba(255, 255, 255, 0.8);

  margin-top: 2px;

}

  

.dropdown-divider {

  height: 1px;

  background-color: var(--border-color);

  margin: 0;

}

  

.logout-button {

  width: 100%;

  padding: 16px 20px;

  text-align: left;

  background: none;

  border: none;

  cursor: pointer;

  color: var(--error-color);

  font-weight: 500;

  font-size: 1.1rem;

  transition: background-color 0.2s;

}

  

.logout-button:hover {

  background-color: var(--hover-color);

}

Layout.module.css

.layout {

  min-height: 100vh;

  background-color: var(--background-light);

}

  

.mainContent {

  margin-top: 70px;

  padding: 30px;

  background-color: var(--background-light);

  min-height: calc(100vh - 70px);

}

  

/* Only apply margin-left when SideNav is present */

.layout:has(> .sideNav) .mainContent {

  margin-left: 250px;

}

  

@media (max-width: 768px) {

  .mainContent {

    margin-left: 0;

    padding: 20px;

  }

}