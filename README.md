[29/03, 11:36] _ramadan_ •: const BASE_URL = 'https://reqres.in';

export const login = async (email, password) => {
  const response = await fetch(`${BASE_URL}/api/login`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, password })
  });
  return response.json();
};

export const getUsers = async (page) => {
  const response = await fetch(`${BASE_URL}/api/users?page=${page}`);
  return response.json();
};

export const updateUser = async (id, userData) => {
  const response = await fetch(`${BASE_URL}/api/users/${id}`, {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(userData)
  });
  return response.json();
};

export const deleteUser = async (id) => {
  const response = await fetch(`${BASE_URL}/api/users/${id}`, {
    method: 'DELETE'
  });
  return response;
};
[29/03, 11:36] _ramadan_ •: import React, { useState } from 'react';
import { login } from '../api';

function Login({ setToken }) {
  const [email, setEmail] = useState('eve.holt@reqres.in');
  const [password, setPassword] = useState('cityslicka');
  const [error, setError] = useState('');

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      const response = await login(email, password);
      if (response.token) {
        setToken(response.token);
      }
    } catch (err) {
      setError('Login failed');
    }
  };

  return (
    <div>
      <h2>Login</h2>
      <form onSubmit={handleSubmit}>
        <input
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          placeholder="Email"
        />
        <input
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          placeholder="Password"
        />
        <button type="submit">Login</button>
      </form>
      {error && <p style={{ color: 'red' }}>{error}</p>}
    </div>
  );
}

export default Login;
[29/03, 11:37] _ramadan_ •: import React from 'react';

function UserCard({ user, onEdit, onDelete }) {
  return (
    <div style={{ border: '1px solid #ccc', padding: '10px', margin: '10px' }}>
      <img src={user.avatar} alt={`${user.first_name} ${user.last_name}`} />
      <h3>{user.first_name} {user.last_name}</h3>
      <p>{user.email}</p>
      <button onClick={() => onEdit(user)}>Edit</button>
      <button onClick={() => onDelete(user.id)}>Delete</button>
    </div>
  );
}

export default UserCard;
[29/03, 11:37] _ramadan_ •: import React, { useState } from 'react';
import { updateUser } from '../api';

function EditUserForm({ user, onSave, onCancel }) {
  const [formData, setFormData] = useState({
    first_name: user.first_name,
    last_name: user.last_name,
    email: user.email
  });
  const [message, setMessage] = useState('');

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      await updateUser(user.id, formData);
      setMessage('User updated successfully');
      onSave({ ...user, ...formData });
    } catch (err) {
      setMessage('Failed to update user');
    }
  };

  return (
    <div>
      <h2>Edit User</h2>
      <form onSubmit={handleSubmit}>
        <input
          value={formData.first_name}
          onChange={(e) => setFormData({ ...formData, first_name: e.target.value })}
          placeholder="First Name"
        />
        <input
          value={formData.last_name}
          onChange={(e) => setFormData({ ...formData, last_name: e.target.value })}
          placeholder="Last Name"
        />
        <input
          value={formData.email}
          onChange={(e) => setFormData({ ...formData, email: e.target.value })}
          placeholder="Email"
        />
        <button type="submit">Save</button>
        <button type="button" onClick={onCancel}>Cancel</button>
      </form>
      {message && <p>{message}</p>}
    </div>
  );
}

export default EditUserForm;
[29/03, 11:38] _ramadan_ •: import React, { useState, useEffect } from 'react';
import { getUsers, deleteUser } from '../api';
import UserCard from './UserCard';
import EditUserForm from './EditUserForm';

function UserList() {
  const [users, setUsers] = useState([]);
  const [page, setPage] = useState(1);
  const [totalPages, setTotalPages] = useState(0);
  const [editingUser, setEditingUser] = useState(null);

  useEffect(() => {
    fetchUsers();
  }, [page]);

  const fetchUsers = async () => {
    const data = await getUsers(page);
    setUsers(data.data);
    setTotalPages(data.total_pages);
  };

  const handleDelete = async (id) => {
    try {
      await deleteUser(id);
      setUsers(users.filter(user => user.id !== id));
    } catch (err) {
      console.error('Failed to delete user');
    }
  };

  const handleUpdate = (updatedUser) => {
    setUsers(users.map(user => 
      user.id === updatedUser.id ? updatedUser : user
    ));
    setEditingUser(null);
  };

  return (
    <div>
      <h2>Users List</h2>
      {editingUser ? (
        <EditUserForm
          user={editingUser}
          onSave={handleUpdate}
          onCancel={() => setEditingUser(null)}
        />
      ) : (
        <>
          <div style={{ display: 'flex', flexWrap: 'wrap' }}>
            {users.map(user => (
              <UserCard
                key={user.id}
                user={user}
                onEdit={setEditingUser}
                onDelete={handleDelete}
              />
            ))}
          </div>
          <div>
            <button
              onClick={() => setPage(page - 1)}
              disabled={page === 1}
            >
              Previous
            </button>
            <span> Page {page} of {totalPages} </span>
            <button
              onClick={() => setPage(page + 1)}
              disabled={page === totalPages}
            >
              Next
            </button>
          </div>
        </>
      )}
    </div>
  );
}

export default UserList;
[29/03, 11:39] _ramadan_ •: import React, { useState } from 'react';
import Login from './components/Login';
import UserList from './components/UserList';

function App() {
  const [token, setToken] = useState(null);

  return (
    <div className="App">
      {!token ? (
        <Login setToken={setToken} />
      ) : (
        <UserList />
      )}
    </div>
  );
}

export default App;
