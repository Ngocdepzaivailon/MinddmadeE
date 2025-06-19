# MinddmadeE
mindmate/
├── public/
│   ├── index.html
│   ├── favicon.ico
│   └── assets/
│       ├── images/
│       └── fonts/
├── src/
│   ├── components/
│   │   ├── Chatbot/
│   │   ├── MoodTracker/
│   │   ├── Resources/
│   │   ├── Assessments/
│   │   ├── Activities/
│   │   └── Shared/
│   ├── pages/
│   │   ├── Home/
│   │   ├── Dashboard/
│   │   ├── Journal/
│   │   ├── Therapy/
│   │   └── Settings/
│   ├── services/
│   ├── styles/
│   ├── App.js
│   ├── index.js
│   └── theme.js
├── package.json
└── README.md
import React, { useState, useEffect } from 'react';
import { ThemeProvider } from '@mui/material/styles';
import { CssBaseline } from '@mui/material';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import { theme } from './theme';
import HomePage from './pages/Home/HomePage';
import DashboardPage from './pages/Dashboard/DashboardPage';
import JournalPage from './pages/Journal/JournalPage';
import TherapyPage from './pages/Therapy/TherapyPage';
import SettingsPage from './pages/Settings/SettingsPage';
import NavBar from './components/Shared/NavBar';
import Sidebar from './components/Shared/Sidebar';
import AuthProvider from './services/AuthProvider';
import ProtectedRoute from './components/Shared/ProtectedRoute';
import { getUserData } from './services/userService';
import { MoodProvider } from './contexts/MoodContext';

function App() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [sidebarOpen, setSidebarOpen] = useState(false);

  useEffect(() => {
    const fetchUser = async () => {
      try {
        const userData = await getUserData();
        setUser(userData);
      } catch (error) {
        console.error('Error fetching user data:', error);
      } finally {
        setLoading(false);
      }
    };

    fetchUser();
  }, []);

  const toggleSidebar = () => {
    setSidebarOpen(!sidebarOpen);
  };

  if (loading) {
    return <div>Loading...</div>;
  }

  return (
    <ThemeProvider theme={theme}>
      <CssBaseline />
      <AuthProvider>
        <MoodProvider>
          <Router>
            <div className="app-container">
              <NavBar toggleSidebar={toggleSidebar} user={user} />
              <Sidebar open={sidebarOpen} onClose={toggleSidebar} />
              <main className="main-content">
                <Routes>
                  <Route path="/" element={<HomePage />} />
                  <Route
                    path="/dashboard"
                    element={
                      <ProtectedRoute>
                        <DashboardPage />
                      </ProtectedRoute>
                    }
                  />
                  <Route
                    path="/journal"
                    element={
                      <ProtectedRoute>
                        <JournalPage />
                      </ProtectedRoute>
                    }
                  />
                  <Route
                    path="/therapy"
                    element={
                      <ProtectedRoute>
                        <TherapyPage />
                      </ProtectedRoute>
                    }
                  />
                  <Route
                    path="/settings"
                    element={
                      <ProtectedRoute>
                        <SettingsPage />
                      </ProtectedRoute>
                    }
                  />
                </Routes>
              </main>
            </div>
          </Router>
        </MoodProvider>
      </AuthProvider>
    </ThemeProvider>
  );
}

export default App;
import React, { useState, useEffect, useRef } from 'react';
import { 
  Box, 
  Avatar, 
  TextField, 
  IconButton, 
  Paper, 
  Typography, 
  List, 
  ListItem, 
  ListItemAvatar, 
  ListItemText, 
  Divider,
  CircularProgress,
  Tooltip
} from '@mui/material';
import { Send, Mood, Psychology, SelfImprovement, MedicalServices } from '@mui/icons-material';
import { styled } from '@mui/system';
import { processMessage } from '../../services/chatbotService';
import { useTheme } from '@mui/material/styles';

const ChatContainer = styled(Box)(({ theme }) => ({
  height: '100%',
  display: 'flex',
  flexDirection: 'column',
  backgroundColor: theme.palette.background.default,
  borderRadius: theme.shape.borderRadius,
  boxShadow: theme.shadows[3],
  overflow: 'hidden',
}));

const MessagesContainer = styled(Box)({
  flex: 1,
  overflowY: 'auto',
  padding: '16px',
});

const InputContainer = styled(Box)({
  padding: '16px',
  borderTop: '1px solid rgba(0, 0, 0, 0.12)',
});

const BotMessage = styled(ListItem)(({ theme }) => ({
  alignItems: 'flex-start',
  padding: '8px 16px',
}));

const UserMessage = styled(ListItem)(({ theme }) => ({
  justifyContent: 'flex-end',
  padding: '8px 16px',
}));

const MessageText = styled(ListItemText)(({ theme, isBot }) => ({
  backgroundColor: isBot ? theme.palette.primary.light : theme.palette.secondary.light,
  color: isBot ? theme.palette.primary.contrastText : theme.palette.secondary.contrastText,
  borderRadius: '18px',
  padding: '8px 16px',
  maxWidth: '70%',
  wordWrap: 'break-word',
}));

const QuickReplies = styled(Box)({
  display: 'flex',
  flexWrap: 'wrap',
  gap: '8px',
  marginBottom: '16px',
});

const QuickReplyButton = styled(IconButton)(({ theme }) => ({
  backgroundColor: theme.palette.background.paper,
  borderRadius: '20px',
  padding: '8px 16px',
  margin: '4px',
  boxShadow: theme.shadows[1],
  '&:hover': {
    backgroundColor: theme.palette.action.hover,
  },
}));

const Chatbot = () => {
  const theme = useTheme();
  const [messages, setMessages] = useState([
    {
      id: 1,
      text: 'Xin chào! Mình là Mindie - trợ lý ảo của bạn. Mình ở đây để lắng nghe và hỗ trợ bạn bất cứ khi nào bạn cần. Hôm nay bạn cảm thấy thế nào?',
      isBot: true,
      timestamp: new Date(),
    }
  ]);
  const [input, setInput] = useState('');
  const [isTyping, setIsTyping] = useState(false);
  const messagesEndRef = useRef(null);

  const quickReplies = [
    { text: 'Mình cảm thấy buồn', icon: <Mood color="primary" /> },
    { text: 'Mình đang lo lắng', icon: <Psychology color="primary" /> },
    { text: 'Cần thư giãn', icon: <SelfImprovement color="primary" /> },
    { text: 'Cần giúp đỡ khẩn cấp', icon: <MedicalServices color="error" /> },
  ];

  useEffect(() => {
    scrollToBottom();
  }, [messages]);

  const scrollToBottom = () => {
    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  };

  const handleSendMessage = async () => {
    if (!input.trim()) return;

    const userMessage = {
      id: messages.length + 1,
      text: input,
      isBot: false,
      timestamp: new Date(),
    };

    setMessages([...messages, userMessage]);
    setInput('');
    setIsTyping(true);

    try {
      const botResponse = await processMessage(input);
      const botMessage = {
        id: messages.length + 2,
        text: botResponse,
        isBot: true,
        timestamp: new Date(),
      };
      setMessages(prev => [...prev, botMessage]);
    } catch (error) {
      console.error('Error processing message:', error);
      const errorMessage = {
        id: messages.length + 2,
        text: 'Xin lỗi, mình gặp chút sự cố. Bạn có thể thử lại hoặc nói theo cách khác được không?',
        isBot: true,
        timestamp: new Date(),
      };
      setMessages(prev => [...prev, errorMessage]);
    } finally {
      setIsTyping(false);
    }
  };

  const handleQuickReply = (reply) => {
    setInput(reply);
  };

  const handleKeyPress = (e) => {
    if (e.key === 'Enter' && !e.shiftKey) {
      e.preventDefault();
      handleSendMessage();
    }
  };

  return (
    <ChatContainer>
      <MessagesContainer>
        <List>
          {messages.map((message) => (
            <React.Fragment key={message.id}>
              {message.isBot ? (
                <BotMessage>
                  <ListItemAvatar>
                    <Avatar src="/assets/images/mindie-avatar.png" alt="Mindie" />
                  </ListItemAvatar>
                  <MessageText 
                    primary={message.text} 
                    secondary={message.timestamp.toLocaleTimeString()} 
                    isBot={true}
                  />
                </BotMessage>
              ) : (
                <UserMessage>
                  <MessageText 
                    primary={message.text} 
                    secondary={message.timestamp.toLocaleTimeString()} 
                    isBot={false}
                  />
                  <ListItemAvatar>
                    <Avatar>B</Avatar>
                  </ListItemAvatar>
                </UserMessage>
              )}
              <Divider variant="inset" component="li" />
            </React.Fragment>
          ))}
          {isTyping && (
            <BotMessage>
              <ListItemAvatar>
                <Avatar src="/assets/images/mindie-avatar.png" alt="Mindie" />
              </ListItemAvatar>
              <Box display="flex" alignItems="center">
                <CircularProgress size={24} />
                <Typography variant="body1" style={{ marginLeft: '8px' }}>Mindie đang trả lời...</Typography>
              </Box>
            </BotMessage>
          )}
          <div ref={messagesEndRef} />
        </List>
      </MessagesContainer>
      
      <QuickReplies>
        {quickReplies.map((reply, index) => (
          <Tooltip title={reply.text} key={index}>
            <QuickReplyButton onClick={() => handleQuickReply(reply.text)}>
              {reply.icon}
            </QuickReplyButton>
          </Tooltip>
        ))}
      </QuickReplies>
      
      <InputContainer>
        <Box display="flex" alignItems="center">
          <TextField
            fullWidth
            variant="outlined"
            placeholder="Nhắn tin với Mindie..."
            value={input}
            onChange={(e) => setInput(e.target.value)}
            onKeyPress={handleKeyPress}
            multiline
            maxRows={4}
          />
          <IconButton 
            color="primary" 
            onClick={handleSendMessage}
            disabled={!input.trim()}
            style={{ marginLeft: '8px' }}
          >
            <Send />
          </IconButton>
        </Box>
      </InputContainer>
    </ChatContainer>
  );
};

export default Chatbot;
import React, { useState, useEffect } from 'react';
import { 
  Box, 
  Typography, 
  Grid, 
  Paper, 
  Button, 
  IconButton, 
  Divider,
  TextField,
  FormControl,
  InputLabel,
  Select,
  MenuItem,
  Slider,
  Avatar
} from '@mui/material';
import { 
  Mood, 
  SentimentVerySatisfied, 
  SentimentSatisfied, 
  SentimentNeutral, 
  SentimentDissatisfied, 
  SentimentVeryDissatisfied,
  Add,
  Remove,
  CalendarToday,
  BarChart,
  ShowChart,
  InsertEmoticon
} from '@mui/icons-material';
import { styled } from '@mui/system';
import { DatePicker } from '@mui/x-date-pickers/DatePicker';
import { AdapterDateFns } from '@mui/x-date-pickers/AdapterDateFns';
import { LocalizationProvider } from '@mui/x-date-pickers/LocalizationProvider';
import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, Legend, ResponsiveContainer } from 'recharts';
import vi from 'date-fns/locale/vi';

const MoodTrackerContainer = styled(Paper)(({ theme }) => ({
  padding: theme.spacing(3),
  borderRadius: theme.shape.borderRadius,
  boxShadow: theme.shadows[3],
}));

const MoodOption = styled(IconButton)(({ theme, selected }) => ({
  border: selected ? `2px solid ${theme.palette.primary.main}` : '2px solid transparent',
  padding: theme.spacing(2),
  margin: theme.spacing(1),
  transition: 'all 0.3s ease',
  '&:hover': {
    transform: 'scale(1.1)',
  },
}));

const MoodIntensitySlider = styled(Slider)(({ theme }) => ({
  width: '80%',
  margin: '0 auto',
  color: theme.palette.primary.main,
}));

const MoodHistoryItem = styled(Paper)(({ theme }) => ({
  padding: theme.spacing(2),
  marginBottom: theme.spacing(2),
  borderRadius: theme.shape.borderRadius,
  display: 'flex',
  alignItems: 'center',
  justifyContent: 'space-between',
}));

const MoodTracker = () => {
  const [selectedMood, setSelectedMood] = useState(null);
  const [intensity, setIntensity] = useState(5);
  const [notes, setNotes] = useState('');
  const [date, setDate] = useState(new Date());
  const [moodHistory, setMoodHistory] = useState([]);
  const [view, setView] = useState('today'); // 'today', 'week', 'month'
  const [trigger, setTrigger] = useState('');
  const [customTrigger, setCustomTrigger] = useState('');

  const moodOptions = [
    { value: 5, icon: <SentimentVerySatisfied fontSize="large" color="success" />, label: 'Rất vui' },
    { value: 4, icon: <SentimentSatisfied fontSize="large" color="success" />, label: 'Vui' },
    { value: 3, icon: <SentimentNeutral fontSize="large" color="warning" />, label: 'Bình thường' },
    { value: 2, icon: <SentimentDissatisfied fontSize="large" color="error" />, label: 'Buồn' },
    { value: 1, icon: <SentimentVeryDissatisfied fontSize="large" color="error" />, label: 'Rất buồn' },
  ];

  const commonTriggers = [
    'Gia đình', 'Bạn bè', 'Công việc', 'Học tập', 'Tình yêu', 'Sức khỏe', 'Tài chính'
  ];

  useEffect(() => {
    // Load mock data for demonstration
    const mockData = generateMockMoodData();
    setMoodHistory(mockData);
  }, []);

  const generateMockMoodData = () => {
    const moods = [];
    const today = new Date();
    
    for (let i = 0; i < 30; i++) {
      const date = new Date();
      date.setDate(today.getDate() - i);
      
      moods.push({
        id: i,
        date: date,
        moodValue: Math.floor(Math.random() * 5) + 1,
        intensity: Math.floor(Math.random() * 10) + 1,
        notes: i % 3 === 0 ? 'Ghi chú mẫu về tâm trạng ngày hôm đó' : '',
        trigger: commonTriggers[Math.floor(Math.random() * commonTriggers.length)]
      });
    }
    
    return moods;
  };

  const handleMoodSelect = (value) => {
    setSelectedMood(value);
  };

  const handleIntensityChange = (event, newValue) => {
    setIntensity(newValue);
  };

  const handleSubmit = () => {
    if (!selectedMood) return;
    
    const newEntry = {
      id: moodHistory.length + 1,
      date: date,
      moodValue: selectedMood,
      intensity: intensity,
      notes: notes,
      trigger: customTrigger || trigger
    };
    
    setMoodHistory([newEntry, ...moodHistory]);
    setSelectedMood(null);
    setIntensity(5);
    setNotes('');
    setTrigger('');
    setCustomTrigger('');
  };

  const getFilteredData = () => {
    const now = new Date();
    
    return moodHistory.filter(entry => {
      const entryDate = new Date(entry.date);
      
      if (view === 'today') {
        return entryDate.toDateString() === now.toDateString();
      } else if (view === 'week') {
        const oneWeekAgo = new Date();
        oneWeekAgo.setDate(now.getDate() - 7);
        return entryDate >= oneWeekAgo;
      } else {
        return true; // month or all
      }
    }).sort((a, b) => new Date(b.date) - new Date(a.date));
  };

  const getChartData = () => {
    const filtered = getFilteredData();
    
    // Group by date for chart
    const grouped = filtered.reduce((acc, entry) => {
      const dateStr = entry.date.toLocaleDateString();
      if (!acc[dateStr]) {
        acc[dateStr] = {
          date: dateStr,
          moodValue: 0,
          intensity: 0,
          count: 0
        };
      }
      acc[dateStr].moodValue += entry.moodValue;
      acc[dateStr].intensity += entry.intensity;
      acc[dateStr].count += 1;
      return acc;
    }, {});
    
    return Object.values(grouped).map(item => ({
      date: item.date,
      moodValue: (item.moodValue / item.count).toFixed(1),
      intensity: (item.intensity / item.count).toFixed(1)
    }));
  };

  const getMoodIcon = (value) => {
    return moodOptions.find(opt => opt.value === value)?.icon || <Mood />;
  };

  return (
    <MoodTrackerContainer>
      <Typography variant="h5" gutterBottom>
        <InsertEmoticon color="primary" style={{ verticalAlign: 'middle', marginRight: '8px' }} />
        Nhật ký cảm xúc
      </Typography>
      
      <Grid container spacing={3}>
        <Grid item xs={12} md={6}>
          <Paper elevation={2} style={{ padding: '16px', marginBottom: '16px' }}>
            <Typography variant="h6" gutterBottom>
              Hôm nay bạn cảm thấy thế nào?
            </Typography>
            
            <Box display="flex" justifyContent="center" mb={3}>
              {moodOptions.map((option) => (
                <MoodOption
                  key={option.value}
                  selected={selectedMood === option.value}
                  onClick={() => handleMoodSelect(option.value)}
                  title={option.label}
                >
                  {option.icon}
                </MoodOption>
              ))}
            </Box>
            
            {selectedMood && (
              <>
                <Typography gutterBottom>
                  Cường độ: {intensity}/10
                </Typography>
                <MoodIntensitySlider
                  value={intensity}
                  onChange={handleIntensityChange}
                  min={1}
                  max={10}
                  step={1}
                  marks={[
                    { value: 1, label: 'Nhẹ' },
                    { value: 10, label: 'Mạnh' }
                  ]}
                />
                
                <LocalizationProvider dateAdapter={AdapterDateFns} adapterLocale={vi}>
                  <DatePicker
                    label="Ngày"
                    value={date}
                    onChange={(newValue) => setDate(newValue)}
                    renderInput={(params) => <TextField {...params} fullWidth margin="normal" />}
                  />
                </LocalizationProvider>
                
                <FormControl fullWidth margin="normal">
                  <InputLabel>Nguyên nhân</InputLabel>
                  <Select
                    value={trigger}
                    onChange={(e) => setTrigger(e.target.value)}
                    label="Nguyên nhân"
                  >
                    <MenuItem value="">
                      <em>Không chọn</em>
                    </MenuItem>
                    {commonTriggers.map((trigger, index) => (
                      <MenuItem key={index} value={trigger}>{trigger}</MenuItem>
                    ))}
                    <MenuItem value="other">Khác</MenuItem>
                  </Select>
                </FormControl>
                
                {trigger === 'other' && (
                  <TextField
                    fullWidth
                    margin="normal"
                    label="Nguyên nhân cụ thể"
                    value={customTrigger}
                    onChange={(e) => setCustomTrigger(e.target.value)}
                  />
                )}
                
                <TextField
                  fullWidth
                  margin="normal"
                  label="Ghi chú (tùy chọn)"
                  multiline
                  rows={3}
                  value={notes}
                  onChange={(e) => setNotes(e.target.value)}
                  placeholder="Bạn muốn ghi chú điều gì về tâm trạng hôm nay?"
                />
                
                <Box mt={2} display="flex" justifyContent="flex-end">
                  <Button 
                    variant="contained" 
                    color="primary" 
                    onClick={handleSubmit}
                    disabled={!selectedMood}
                  >
                    Lưu cảm xúc
                  </Button>
                </Box>
              </>
            )}
          </Paper>
        </Grid>
        
        <Grid item xs={12} md={6}>
          <Paper elevation={2} style={{ padding: '16px', marginBottom: '16px' }}>
            <Box display="flex" justifyContent="space-between" alignItems="center" mb={2}>
              <Typography variant="h6">Biểu đồ cảm xúc</Typography>
              <Box>
                <Button 
                  size="small" 
                  color={view === 'today' ? 'primary' : 'default'} 
                  onClick={() => setView('today')}
                  startIcon={<CalendarToday />}
                >
                  Hôm nay
                </Button>
                <Button 
                  size="small" 
                  color={view === 'week' ? 'primary' : 'default'} 
                  onClick={() => setView('week')}
                  startIcon={<BarChart />}
                >
                  1 tuần
                </Button>
                <Button 
                  size="small" 
                  color={view === 'month' ? 'primary' : 'default'} 
                  onClick={() => setView('month')}
                  startIcon={<ShowChart />}
                >
                  1 tháng
                </Button>
              </Box>
            </Box>
            
            <div style={{ height: '300px' }}>
              <ResponsiveContainer width="100%" height="100%">
                <LineChart data={getChartData()}>
                  <CartesianGrid strokeDasharray="3 3" />
                  <XAxis dataKey="date" />
                  <YAxis domain={[0, 5]} />
                  <Tooltip />
                  <Legend />
                  <Line 
                    type="monotone" 
                    dataKey="moodValue" 
                    name="Tâm trạng (1-5)" 
                    stroke="#8884d8" 
                    activeDot={{ r: 8 }} 
                  />
                  <Line 
                    type="monotone" 
                    dataKey="intensity" 
                    name="Cường độ (1-10)" 
                    stroke="#82ca9d" 
                  />
                </LineChart>
              </ResponsiveContainer>
            </div>
          </Paper>
          
          <Paper elevation={2} style={{ padding: '16px' }}>
            <Typography variant="h6" gutterBottom>
              Lịch sử cảm xúc
            </Typography>
            
            {getFilteredData().length === 0 ? (
              <Typography variant="body1" color="textSecondary" align="center" style={{ padding: '16px' }}>
                Chưa có dữ liệu. Hãy ghi lại cảm xúc của bạn!
              </Typography>
            ) : (
              <Box>
                {getFilteredData().map((entry) => (
                  <MoodHistoryItem key={entry.id} elevation={1}>
                    <Box display="flex" alignItems="center">
                      <Avatar style={{ marginRight: '16px' }}>
                        {getMoodIcon(entry.moodValue)}
                      </Avatar>
                      <Box>
                        <Typography variant="body1">
                          {entry.date.toLocaleDateString()} - {entry.date.toLocaleTimeString()}
                        </Typography>
                        <Typography variant="body2" color="textSecondary">
                          {entry.trigger && `Nguyên nhân: ${entry.trigger}`}
                        </Typography>
                        {entry.notes && (
                          <Typography variant="body2" style={{ marginTop: '4px' }}>
                            {entry.notes}
                          </Typography>
                        )}
                      </Box>
                    </Box>
                    <Box textAlign="right">
                      <Typography variant="body1">
                        {moodOptions.find(opt => opt.value === entry.moodValue)?.label}
                      </Typography>
                      <Typography variant="body2" color="textSecondary">
                        Cường độ: {entry.intensity}/10
                      </Typography>
                    </Box>
                  </MoodHistoryItem>
                ))}
              </Box>
            )}
          </Paper>
        </Grid>
      </Grid>
    </MoodTrackerContainer>
  );
};

export default MoodTracker;
import { createTheme } from '@mui/material/styles';

const theme = createTheme({
  palette: {
    primary: {
      main: '#6a1b9a',
      light: '#9c4dcc',
      dark: '#38006b',
      contrastText: '#ffffff',
    },
    secondary: {
      main: '#ff6d00',
      light: '#ff9e40',
      dark: '#c43c00',
      contrastText: '#ffffff',
    },
    background: {
      default: '#f5f5f5',
      paper: '#ffffff',
    },
    text: {
      primary: '#212121',
      secondary: '#757575',
    },
    error: {
      main: '#d32f2f',
    },
    warning: {
      main: '#ffa000',
    },
    info: {
      main: '#1976d2',
    },
    success: {
      main: '#388e3c',
    },
  },
  typography: {
    fontFamily: [
      '"Segoe UI"',
      'Roboto',
      '"Helvetica Neue"',
      'Arial',
      'sans-serif',
      '"Apple Color Emoji"',
      '"Segoe UI Emoji"',
      '"Segoe UI Symbol"',
    ].join(','),
    h1: {
      fontSize: '2.5rem',
      fontWeight: 500,
    },
    h2: {
      fontSize: '2rem',
      fontWeight: 500,
    },
    h3: {
      fontSize: '1.75rem',
      fontWeight: 500,
    },
    h4: {
      fontSize: '1.5rem',
      fontWeight: 500,
    },
    h5: {
      fontSize: '1.25rem',
      fontWeight: 500,
    },
    h6: {
      fontSize: '1rem',
      fontWeight: 500,
    },
    body1: {
      fontSize: '1rem',
    },
    body2: {
      fontSize: '0.875rem',
    },
    button: {
      textTransform: 'none',
    },
  },
  shape: {
    borderRadius: 8,
  },
  components: {
    MuiButton: {
      styleOverrides: {
        root: {
          borderRadius: 20,
          padding: '8px 16px',
        },
      },
    },
    MuiCard: {
      styleOverrides: {
        root: {
          borderRadius: 12,
          boxShadow: '0 4px 12px rgba(0, 0, 0, 0.1)',
          transition: 'box-shadow 0.3s ease-in-out',
          '&:hover': {
            boxShadow: '0 8px 16px rgba(0, 0, 0, 0.2)',
          },
        },
      },
    },
    MuiTextField: {
      styleOverrides: {
        root: {
          '& .MuiOutlinedInput-root': {
            borderRadius: 12,
          },
        },
      },
    },
  },
});

export default theme;
import axios from 'axios';

const API_URL = process.env.REACT_APP_CHATBOT_API_URL || 'https://api.mindmate.ai/v1/chat';

const responses = {
  greetings: [
    "Xin chào! Mình là Mindie, luôn sẵn sàng lắng nghe bạn.",
    "Chào bạn! Mình ở đây để hỗ trợ bạn bất cứ khi nào bạn cần.",
    "Hi! Mình là Mindie, người bạn đồng hành tâm lý của bạn."
  ],
  feelings: {
    happy: [
      "Thật tuyệt khi bạn đang cảm thấy vui! Bạn muốn chia sẻ thêm điều gì làm bạn hạnh phúc không?",
      "Vui quá! Những khoảnh khắc như thế này thật đáng trân trọng. Bạn đang làm gì khi cảm thấy vui thế?"
    ],
    sad: [
      "Mình rất tiếc khi nghe bạn đang buồn. Bạn có muốn chia sẻ thêm không? Mình luôn ở đây để lắng nghe.",
      "Buồn là cảm xúc tự nhiên mà ai cũng trải qua. Hãy nhớ rằng bạn không cô đơn. Bạn có muốn nói thêm về điều gì khiến bạn buồn không?"
    ],
    anxious: [
      "Lo lắng có thể rất khó chịu. Bạn có thể thử hít thở sâu vài lần. Bạn muốn chia sẻ điều gì đang làm bạn lo lắng không?",
      "Mình hiểu cảm giác lo lắng có thể rất áp lực. Đôi khi việc nói ra những điều khiến bạn lo lắng có thể giúp giảm bớt căng thẳng."
    ],
    angry: [
      "Tức giận là cảm xúc hoàn toàn bình thường. Bạn có thể thử đếm từ 1 đến 10 hoặc đi dạo một chút. Bạn có muốn nói về điều gì khiến bạn tức giận không?",
      "Mình hiểu bạn đang rất tức giận. Hãy nhớ rằng cảm xúc này rồi sẽ qua. Bạn muốn chia sẻ thêm không?"
    ]
  },
  coping: [
    "Bạn có thể thử một số kỹ thuật thư giãn như hít thở sâu, thiền hoặc nghe nhạc nhẹ.",
    "Đôi khi viết ra những suy nghĩ của mình có thể giúp bạn cảm thấy nhẹ nhõm hơn.",
    "Tập thể dục nhẹ nhàng như đi bộ có thể giúp cải thiện tâm trạng đấy."
  ],
  resources: [
    "Mình có một số bài viết và video về chủ đề này, bạn có muốn mình gửi cho bạn không?",
    "Mình có thể giới thiệu cho bạn một số tài nguyên hữu ích về vấn đề này."
  ],
  emergency: [
    "Nghe có vẻ bạn đang gặp khủng hoảng nghiêm trọng. Bạn có thể liên hệ ngay với đường dây nóng hỗ trợ tâm lý 111 hoặc đến cơ sở y tế gần nhất.",
    "Mình rất lo lắng cho bạn. Bạn có thể liên hệ với người thân hoặc chuyên gia ngay lập tức. Bạn không cô đơn trong cuộc chiến này."
  ]
};

const detectIntent = (message) => {
  const lowerMsg = message.toLowerCase();
  
  // Check for greetings
  if (/xin chào|hello|hi|chào|helo/.test(lowerMsg)) {
    return 'greetings';
  }
  
  // Check for feelings
  if (/buồn|tệ|tồi tệ|chán nản|thất vọng|khóc/.test(lowerMsg)) {
    return 'sad';
  }
  if (/vui|hạnh phúc|phấn khởi|tuyệt vời|hào hứng/.test(lowerMsg)) {
    return 'happy';
  }
  if (/lo lắng|lo âu|hoảng loạn|hồi hộp|sợ hãi|bất an/.test(lowerMsg)) {
    return 'anxious';
  }
  if (/tức giận|giận|bực|phát điên|khó chịu|ức chế/.test(lowerMsg)) {
    return 'angry';
  }
  
  // Check for emergency
  if (/tự tử|chết|kết thúc|mệt mỏi|không muốn sống/.test(lowerMsg)) {
    return 'emergency';
  }
  
  // Check for coping requests
  if (/làm sao|phải làm gì|cách nào|giúp|gợi ý/.test(lowerMsg)) {
    return 'coping';
  }
  
  // Default to general response
  return 'general';
};

const getRandomResponse = (responseType) => {
  const responseArray = responses[responseType];
  return responseArray[Math.floor(Math.random() * responseArray.length)];
};

export const processMessage = async (message) => {
  try {
    // First try to get response from local database
    const intent = detectIntent(message);
    
    if (intent === 'emergency') {
      return getRandomResponse('emergency');
    }
    
    // For more complex queries, use the API
    const response = await axios.post(API_URL, {
      message: message,
      context: 'teen_mental_health',
      language: 'vi'
    });
    
    return response.data.reply || getRandomResponse(intent);
  } catch (error) {
    console.error('Error processing message with API:', error);
    // Fallback to local responses
    const intent = detectIntent(message);
    return getRandomResponse(intent);
  }
};
