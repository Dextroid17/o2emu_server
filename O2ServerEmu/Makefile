CXX := g++
CXXFLAGS := -I/home/dextroid17/local/boost
LDFLAGS := -L/home/dextroid17/local/boost/lib -lboost_system -lboost_thread -lpthread

SRCS := $(wildcard *.cpp)
OBJS := $(patsubst %.cpp,%.o,$(SRCS))

TARGET=O2ServerEmu

O2ServerEmu: $(OBJS)
	$(CXX) -o $@ $^ $(CXXFLAGS) $(LDFLAGS)

%.o : %.cpp
	$(CXX) $(CXXFLAGS) -c $<

clean:
	rm -f $(TARGET) *.o
