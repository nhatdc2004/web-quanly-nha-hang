package com.manage.restaurant.service.impl;

import com.manage.restaurant.model.User;
import com.manage.restaurant.repository.UserRepository;
import com.manage.restaurant.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.Optional;

@Service
@Transactional
public class UserServiceImpl implements UserService {

    @Autowired
    private UserRepository userRepository;

    @Override
    public List<User> getAllUsers() {
        return userRepository.findAll();
    }

    @Override
    public Optional<User> getUserById(Long id) {
        return userRepository.findById(id);
    }

    @Override
    public User saveUser(User user) {
        return userRepository.save(user);
    }

    @Override
    public User updateUser(Long id, User user) {
        if (userRepository.existsById(id)) {
            user.setId(id);
            return userRepository.save(user);
        }
        throw new RuntimeException("User not found with id: " + id);
    }

    @Override
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }

    @Override
    public Optional<User> getUserByEmail(String email) {
        return userRepository.findByEmail(email);
    }

    @Override
    public Optional<User> getUserByPhoneNumber(String phoneNumber) {
        return userRepository.findByPhoneNumber(phoneNumber);
    }

    @Override
    public List<User> getUsersByRole(String role) {
        return userRepository.findByRole(role);
    }

    @Override
    public boolean existsByEmail(String email) {
        return userRepository.existsByEmail(email);
    }

    @Override
    public boolean existsByPhoneNumber(String phoneNumber) {
        return userRepository.existsByPhoneNumber(phoneNumber);
    }

    @Override
    public User createUser(String name, String email, String phoneNumber, String password, String role) {
        if (existsByEmail(email)) {
            throw new RuntimeException("User already exists with email: " + email);
        }

        // Encode password
        BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
        String encodedPassword = passwordEncoder.encode(password);

        User user = new User();
        user.setName(name);
        user.setEmail(email);
        user.setPhoneNumber(phoneNumber);
        user.setPassword(encodedPassword); // hashed password
        user.setRole(role);

        return userRepository.save(user);
    }

    @Override
    public User createOrGetUser(String name, String email, String phoneNumber) {
        // Try to find existing user by email
        Optional<User> existingUser = getUserByEmail(email);
        if (existingUser.isPresent()) {
            return existingUser.get();
        }
        
        // Try to find existing user by phone number
        existingUser = getUserByPhoneNumber(phoneNumber);
        if (existingUser.isPresent()) {
            return existingUser.get();
        }
        
        // Create new user if not found
        User user = new User();
        user.setName(name);
        user.setEmail(email);
        user.setPhoneNumber(phoneNumber);
        user.setPassword("defaultPassword123"); // Default password for order users
        user.setRole("CUSTOMER"); // Default role for order users
        
        return userRepository.save(user);
    }

    @Override
    public User updateUserInfo(Long id, String name, String email, String phoneNumber) {
        Optional<User> userOptional = userRepository.findById(id);
        if (userOptional.isPresent()) {
            User user = userOptional.get();
            
            // Check if email is being changed and if new email already exists
            if (!user.getEmail().equals(email) && existsByEmail(email)) {
                throw new RuntimeException("Email already exists: " + email);
            }
            
            // Check if phone number is being changed and if new phone number already exists
            if (!user.getPhoneNumber().equals(phoneNumber) && existsByPhoneNumber(phoneNumber)) {
                throw new RuntimeException("Phone number already exists: " + phoneNumber);
            }
            
            user.setName(name);
            user.setEmail(email);
            user.setPhoneNumber(phoneNumber);
            
            return userRepository.save(user);
        }
        throw new RuntimeException("User not found with id: " + id);
    }

    @Override
    public User updateUserPassword(Long id, String newPassword) {
        Optional<User> userOpt = userRepository.findById(id);
        if (userOpt.isPresent()) {
            User user = userOpt.get();
            user.setPassword(newPassword); // In production, this should be hashed
            return userRepository.save(user);
        }
        throw new RuntimeException("User not found with id: " + id);
    }

    @Override
    public User updateUserRole(Long id, String role) {
        Optional<User> userOpt = userRepository.findById(id);
        if (userOpt.isPresent()) {
            User user = userOpt.get();
            user.setRole(role);
            return userRepository.save(user);
        }
        throw new RuntimeException("User not found with id: " + id);
    }

    @Override
    public Optional<User> authenticateUser(String email, String password) {
        Optional<User> userOpt = userRepository.findByEmail(email);
        if (userOpt.isPresent() && userOpt.get().getPassword().equals(password)) {
            return userOpt;
        }
        return Optional.empty();
    }

    @Override
    public boolean validatePassword(String email, String password) {
        return authenticateUser(email, password).isPresent();
    }

    @Override
    public List<User> searchUsersByName(String name) {
        return userRepository.findByNameContainingIgnoreCase(name);
    }

    @Override
    public List<User> searchUsersByEmail(String email) {
        return userRepository.findByEmailContainingIgnoreCase(email);
    }

    @Override
    public List<User> getCustomers() {
        return getUsersByRole("CUSTOMER");
    }

    @Override
    public List<User> getAdmins() {
        return getUsersByRole("ADMIN");
    }

    @Override
    public List<User> getStaff() {
        return getUsersByRole("STAFF");
    }

    @Override
    public long getTotalUserCount() {
        return userRepository.count();
    }

    @Override
    public long getUserCountByRole(String role) {
        return userRepository.countByRole(role);
    }
}
